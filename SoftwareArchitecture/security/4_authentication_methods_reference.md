# Authentication Methods — Reference Guide

This document lists common authentication methods (from the provided screenshot and additional important models), with definitions, key features, examples, pros, cons, and typical use cases. Use this as a quick reference when designing identity and access systems.

---

## Table of contents
1. [Token authentication](#1-token-authentication)
2. [Certificate authentication (X.509)](#2-certificate-authentication-x509)
3. [Passwordless authentication](#3-passwordless-authentication)
4. [Passwords (traditional)](#4-passwords-traditional)
5. [OpenID / OpenID Connect (OIDC)](#5-openid--openid-connect-oidc)
6. [Lightweight Directory Access Protocol (LDAP) authentication](#6-lightweight-directory-access-protocol-ldap-authentication)
7. [Challenge-Handshake Authentication Protocol (CHAP) / challenge-based](#7-challenge-handshake-authentication-protocol-chap--challenge-based-auth)
8. [Fingerprint authentication (biometrics)](#8-fingerprint-authentication-biometrics)
9. [Biometric authentication (general)](#9-biometric-authentication-general)
10. [Single sign-on (SSO)](#10-single-sign-on-sso)
11. [Adaptive (risk-based) authentication](#11-adaptive-risk-based-authentication)
12. [Behavioral authentication](#12-behavioral-authentication)
13. [SAML (Security Assertion Markup Language)](#13-saml-security-assertion-markup-language)
14. [Single-factor authentication (SFA)](#14-single-factor-authentication-sfa)
15. [Continuous authentication](#15-continuous-authentication)
16. [Remote authentication (RADIUS, TACACS+, VPN auth)](#16-remote-authentication-radius-tacacs-vpn-auth)
17. [Two-factor authentication (2FA)](#17-two-factor-authentication-2fa)
18. [Password Authentication Protocol (PAP)](#18-password-authentication-protocol-pap)
19. [User authentication methods (overview)](#19-user-authentication-methods-overview)
20. [OAuth 2.0](#20-oauth-20)
21. [API authentication (keys, HMAC)](#21-api-authentication-keys-hmac-mutual-tls)
22. [EAP (Extensible Authentication Protocol)](#22-eap-extensible-authentication-protocol)
23. [Vault authentication (secrets managers)](#23-vault-authentication-secrets-managers)
24. [RBAC (Role-Based Access Control)](#24-rbac-role-based-access-control--access-control-model)
25. [ABAC (Attribute-Based Access Control)](#25-abac-attribute-based-access-control--access-control-model)
26. [Risk-based authentication (RBA)](#26-risk-based-authentication-rba--risk-adaptive-authentication)
27. [Capability-based authentication](#27-capability-based-authentication-capability-tokens--object-capability-model)
28. [Additional notes and references](#28-additional-notes--references)

---

## 1. Token Authentication

**Definition:** Authentication that relies on tokens (opaque or structured) issued by an authentication service to represent an identity and claims. Tokens are presented to access resources without re-supplying credentials each time.

**Key features:**
- Stateless or stateful depending on design
- Can be short-lived (access tokens) with refresh tokens
- Common token formats: JWT (JSON Web Token), opaque random tokens

**Examples:** JWT access tokens for APIs, session tokens stored in cookies, OAuth 2.0 Bearer tokens.

**Pros:**
- Efficient for APIs and distributed services
- Scales well (stateless tokens reduce server-side session storage)
- Encapsulates claims (JWT) so services can make authorization decisions locally

**Cons:**
- JWTs can be large and, if misconfigured, can leak sensitive claims
- Revocation is harder for stateless tokens (requires token blacklist or short lifetimes)
- Need secure storage & transmission (HTTPS)

**Use cases:** REST APIs, microservices, single-page applications, mobile apps.

---

## 2. Certificate authentication (X.509)

**Definition:** Uses public key certificates (X.509) to authenticate clients and/or servers using asymmetric cryptography and TLS mutual authentication.

**Key features:**
- Strong cryptographic assurance
- Supports mutual TLS (mTLS) for client and server authentication
- Certificates have expiration dates and can be revoked (CRL/OCSP)

**Examples:** Client TLS certificates, smart cards, enterprise PKI for VPN access.

**Pros:**
- Very strong identity proofing when managed properly
- No password to steal
- Integrates with TPM/secure hardware for key protection

**Cons:**
- Operational complexity (provisioning, rotation, revocation)
- Usability friction for users without automated enrollment
- Cost and infrastructure for PKI management

**Use cases:** High-security enterprise environments, IoT device authentication, mutual TLS for microservices.

---

## 3. Passwordless authentication

**Definition:** Auth flow that avoids passwords; uses other factors like magic links, one-time codes to email/phone, or cryptographic keys (WebAuthn).

**Key features:**
- Eliminates password storage and related backbone attacks
- Can use device-bound keys (WebAuthn) or email/sms-based magic links/OTPs
- Better user experience than remembering passwords

**Examples:** WebAuthn with platform authenticator (Touch ID/Windows Hello), magic link via email, FIDO2 hardware keys.

**Pros:**
- Reduces phishing and credential stuffing risks
- Improved security with hardware-bound keys
- Better UX in many flows

**Cons:**
- Dependence on email/SMS or device capabilities (SMS less secure)
- Account recovery flow must be carefully designed
- Adoption effort across devices and browsers for WebAuthn

**Use cases:** Consumer apps prioritizing UX, high-security apps using FIDO2 keys, services wanting to reduce password support costs.

---

## 4. Passwords (traditional)

**Definition:** Secret string known to the user used as the primary authentication factor.

**Key features:**
- Simple and widely understood
- Often combined with hashing and salting on the server
- Vulnerable to brute-force, reuse, and phishing when weak

**Examples:** Username + password login, passphrases.

**Pros:**
- Universal support and low friction to implement
- Familiar UX for users

**Cons:**
- Users choose weak or reused passwords
- Credential stuffing and phishing attacks
- Requires secure hashing, rate limiting, and breach detection

**Use cases:** Legacy systems, low-risk services, or as part of multi-factor schemes.

---

## 5. OpenID / OpenID Connect (OIDC)

**Definition:** OpenID Connect is an identity layer on top of OAuth 2.0 that authenticates users and provides ID tokens (JWT) containing user identity claims.

**Key features:**
- Standardized flows (Authorization Code, Implicit, Hybrid)
- ID tokens + userinfo endpoint for user profile data
- Supports federation and SSO scenarios

**Examples:** Login with Google/Facebook, enterprise OpenID Connect providers.

**Pros:**
- Standardized and interoperable
- Enables single sign-on and federation
- Separation between authentication and application logic

**Cons:**
- Complexity in configuring scopes, claims, and token lifetimes
- Requires secure redirect URI handling to prevent open redirect attacks

**Use cases:** Federated login, single sign-on, integrating third-party identity providers.

---

## 6. Lightweight Directory Access Protocol (LDAP) authentication

**Definition:** Authentication using LDAP directories (e.g., Active Directory) by binding with DN and password or using Kerberos for SSO.

**Key features:**
- Centralized credential store for enterprise users
- Supports group/role lookup for authorization
- Often integrated with Kerberos for SSO on networks

**Examples:** AD authentication for corporate apps, LDAP bind authentication for internal services.

**Pros:**
- Centralized identity management and policy enforcement
- Mature tooling and ecosystem

**Cons:**
- LDAP protocol specifics can be complex
- Less suited for internet-scale public-facing services compared to OAuth/OIDC

**Use cases:** Enterprise intranet apps, legacy systems, services that need AD integration.

---

## 7. Challenge-Handshake Authentication Protocol (CHAP) / challenge-based auth

**Definition:** Challenge-response scheme where the server issues a challenge; the client proves knowledge of secret by returning a response (often hashed), avoiding sending the secret in plaintext.

**Key features:**
- Avoids transmitting raw passwords over the wire
- Variants involve different cryptographic functions
- Can defend against replay if nonces are used

**Examples:** CHAP for PPP connections, Kerberos challenge-response mechanisms, SRP (Secure Remote Password) protocol.

**Pros:**
- Secret not transmitted directly
- Improved security over plaintext protocols

**Cons:**
- Replay and downgrade risks if nonces/timestamps not used properly
- Complexity compared to simple password schemes

**Use cases:** Network authentication (PPP, some VPNs), authentication where password shouldn't be sent over network.

---

## 8. Fingerprint authentication (biometrics)

**Definition:** Uses fingerprint biometric sensors to verify user identity by matching stored biometric templates.

**Key features:**
- Tied to unique user physical attribute
- Usually used as a second factor or local unlock (device-level)
- Modern systems store templates on device (secure enclave) rather than server

**Examples:** Mobile fingerprint unlock, laptop fingerprint readers

**Pros:**
- Convenient and fast for end users
- Harder to reuse compared to passwords

**Cons:**
- Biometrics are not secret (you can’t change your fingerprint)
- Privacy concerns, spoofing risk (high-quality replicas)
- Needs secure template storage and anti-spoofing

**Use cases:** Device unlock, local authentication, optional second factor in consumer apps.

---

## 9. Biometric authentication (general)

**Definition:** Authentication using biological traits: fingerprints, face recognition, iris, voice, or behavioral biometrics.

**Key features:**
- Can be passive or active (continuous behavioral biometrics)
- Usually combined with secure hardware to store templates (TPM, Secure Enclave)
- Privacy and regulatory considerations apply

**Examples:** Face ID, iris scanners at airports, voice biometrics in call centers

**Pros:**
- High convenience and non-repudiation in many contexts
- Good for continuous authentication when behavioral measures used

**Cons:**
- False positives/negatives (tuning required)
- Privacy/regulatory issues (GDPR), potential for surveillance misuse
- Sensor and environment dependent

**Use cases:** Device unlocking, high-security physical access, continuous session validation.

---

## 10. Single sign-on (SSO)

**Definition:** A user authenticates once with an identity provider and can access multiple independent services without re-authenticating.

**Key features:**
- Centralized authentication and session management
- Uses federation standards like SAML or OIDC
- Improves user experience and lowers password entry frequency

**Examples:** Corporate SSO with SAML for SaaS apps, Google Workspace SSO

**Pros:**
- Reduced password fatigue and fewer login prompts
- Centralized access control and auditing

**Cons:**
- Single point of failure if IdP is down
- Compromise of IdP can lead to broad access compromise

**Use cases:** Enterprise application suites, SaaS integrations for organizations.

---

## 11. Adaptive (risk-based) authentication

**Definition:** Authentication that adjusts required assurance level dynamically based on contextual risk signals (device, IP reputation, geolocation, behavior).

**Key features:**
- Risk scoring engine that evaluates each login attempt
- Step-up authentication (require MFA) when risk is high
- Uses signals: device fingerprint, geolocation, time, velocity, IP reputation

**Examples:** Force MFA if login from new country or anonymizing proxy

**Pros:**
- Balances security and usability by prompting farther only when necessary
- Reduces friction for low-risk users

**Cons:**
- Risk models can generate false positives/negatives
- Complexity in tuning rules and integrating signals

**Use cases:** Banking, high-value SaaS, enterprise remote access, any service wanting adaptive MFA.

---

## 12. Behavioral authentication

**Definition:** Uses patterns of user behavior (typing patterns, mouse movement, device usage) to continuously or passively authenticate users.

**Key features:**
- Passive and continuous, can detect anomalies mid-session
- Machine learning models to profile typical user behavior
- Often used as an additional risk signal in adaptive auth

**Examples:** Keystroke dynamics, transaction behavior analysis

**Pros:**
- Low friction for users; can detect account takeover or fraud
- Works continuously to catch session hijacks

**Cons:**
- Privacy concerns and false positive tuning needed
- Requires data collection and ML infrastructure

**Use cases:** Fraud detection in finance, high-risk account monitoring, adaptive auth signal.

---

## 13. SAML (Security Assertion Markup Language)

**Definition:** XML-based standard for exchanging authentication and authorization data between parties, commonly used for enterprise SSO.

**Key features:**
- Assertion-based (IdP issues assertions to SP)
- Uses browser redirection flows and signed assertions
- Strong enterprise integration for SSO

**Examples:** Corporate SSO using ADFS, Okta, OneLogin with SAML to SaaS apps

**Pros:**
- Mature standard for enterprise federation
- Rich attribute assertions and policy control

**Cons:**
- XML complexity and configuration pain
- Heavier than OIDC for modern web apps

**Use cases:** Enterprise SSO, legacy SaaS integrations in corporate environments.

---

## 14. Single-factor authentication (SFA)

**Definition:** Authentication that relies on a single type of evidence (e.g., password or biometric).

**Key features:**
- Simpler UX but lower assurance than multi-factor
- Often used for low-sensitivity actions

**Examples:** Password-only login, PIN-only device unlock

**Pros:**
- Low friction, easy to implement

**Cons:**
- Single point of failure; easily compromised

**Use cases:** Low-risk consumer apps, initial frictionless experiences with later step-up.

---

## 15. Continuous authentication

**Definition:** Ongoing assessment of user authenticity during a session using behavioral signals, device posture, and periodic revalidation.

**Key features:**
- Can silently re-evaluate trust and revoke/step-up on anomalies
- Integrates with behavioral biometrics and device telemetry

**Examples:** Banking app detects abnormal transaction patterns and prompts re-authentication mid-session

**Pros:**
- Detects compromised sessions quickly
- Low user friction if mostly silent

**Cons:**
- Infrastructure and privacy complexity
- Potential for false positives disrupting UX

**Use cases:** High-value transactions, long-lived sessions, privileged admin sessions.

---

## 16. Remote authentication (RADIUS, TACACS+, VPN auth)

**Definition:** Network-oriented authentication protocols where remote devices authenticate to centralized servers.

**Key features:**
- RADIUS for network access and VPN authentication
- TACACS+ commonly used for device/console admin access
- Centralized accounting and policy enforcement

**Examples:** Wi‑Fi 802.1X with RADIUS, network device admin via TACACS+

**Pros:**
- Centralized policy, logging, and MFA integration
- Protocols tailored for network access control

**Cons:**
- Operational complexity, additional infrastructure
- Latency or availability impact can affect access

**Use cases:** Enterprise network access, VPN, network device administration.

---

## 17. Two-factor authentication (2FA)

**Definition:** Requires two different types of evidence (something you know, have, or are).

**Key features:**
- Common combinations: password + OTP, password + hardware token, password + push approval
- Increases assurance compared to single-factor

**Examples:** SMS OTP, authenticator apps (TOTP), hardware tokens (YubiKey), push-based approval

**Pros:**
- Significantly reduces credential-based attacks
- Diverse options for user preference (app, SMS, hardware)

**Cons:**
- SMS is vulnerable to SIM swap and interception
- Users may resist extra steps without clear value

**Use cases:** Account login protection, admin access, sensitive operations (bank transfers).

---

## 18. Password Authentication Protocol (PAP)

**Definition:** Simple protocol that sends username/password in plaintext to server (often within an encrypted tunnel historically).

**Key features:**
- Very basic; insecure over unencrypted channels
- Supplanted by more secure challenge-response and TLS-based methods

**Examples:** Old PPP implementations; rarely used today outside legacy contexts

**Pros:**
- Simple implementation

**Cons:**
- Sends credentials in plaintext; insecure without TLS
- Largely deprecated for modern systems

**Use cases:** Legacy network connections; avoid for new systems.

---

## 19. User authentication methods (overview)

This covers the broad categories (knowledge, possession, inherence, location, and behavioral factors) and combinations thereof to achieve required assurance levels.

- **Knowledge:** password, PIN, secret questions (weak, avoid questions)
- **Possession:** hardware token, phone, certificate
- **Inherence:** biometrics (fingerprint, face)
- **Location/Time:** geofencing, IP range checks
- **Behavioral:** keystroke, mouse, transaction history

Use layered multi-factor and adaptive approaches for stronger security while maintaining user experience.

---

## 20. OAuth 2.0

**Definition:** Authorization framework for delegated access (not strictly authentication). Often paired with OIDC for authentication.

**Key features:**
- Authorization grants (authorization code, client credentials, implicit, resource owner password)
- Access and refresh tokens for resource access
- Widely used for API authorization

**Examples:** Third-party app accessing Google Drive using OAuth 2.0

**Pros:**
- Standardized delegation model
- Flexible flows for different client types

**Cons:**
- Misuse as authentication can lead to security problems (use OIDC for identity)
- Complexity in securely implementing flows

**Use cases:** API authorization, delegated access for third-party apps.

---

## 21. API authentication (keys, HMAC, mutual TLS)

**Definition:** Methods to authenticate and authorize API clients, often using API keys, HMAC signatures, or mTLS client certs.

**Key features:**
- API keys are simple but should be treated as secrets
- HMAC (e.g., AWS Signature v4) ensures request integrity and authenticity
- mTLS provides strong client identity with mutual TLS

**Examples:** API key in header, HMAC-signed requests, client TLS certs

**Pros:**
- HMAC and mTLS offer high assurance for server-to-server auth
- Fine-grained key management and rotation possible

**Cons:**
- API keys can be leaked if not managed properly
- mTLS operational complexity

**Use cases:** Server-to-server APIs, third-party integration, SDK clients.

---

## 22. EAP (Extensible Authentication Protocol)

**Definition:** A framework for network access authentication supporting multiple authentication methods (EAP-TLS, EAP-MSCHAPv2, etc.).

**Key features:**
- Pluggable methods for Wi‑Fi 802.1X authentication and PPP
- Can support certificates, tokens, or other mechanisms

**Examples:** EAP-TLS for certificate-based Wi‑Fi; EAP-MSCHAPv2 for MS environments

**Pros:**
- Flexible, widely supported for network authentication
- Strong methods (EAP-TLS) provide certificate-level security

**Cons:**
- Complexity in configuration and interoperability
- Weaker EAP methods can be exploited if chosen incorrectly

**Use cases:** Enterprise Wi‑Fi, network access control, VPN authentication.

---

## 23. Vault authentication (secrets managers)

**Definition:** Authentication mechanisms provided by secrets management systems (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault) for clients and applications to retrieve secrets.

**Key features:**
- Dynamic credentials, short-lived secrets, leasing/rotation
- Multiple auth methods: AppRole, AWS IAM, TLS cert, Kubernetes auth, token auth

**Examples:** HashiCorp Vault AppRole or Kubernetes service account auth

**Pros:**
- Reduces long-lived secret exposure
- Centralized auditing and rotation

**Cons:**
- Operational dependency on Vault availability
- Complexity in auth method policies and provisioning

**Use cases:** Application identity, automated secret retrieval, cloud-native apps.

---

## 24. RBAC (Role-Based Access Control) — Access Control model

**Definition:** Authorization model where permissions are assigned to roles, and users are assigned roles. Access is determined by user role membership.

**Key features:**
- Roles map to sets of permissions/privileges
- Simplifies permission management in groups
- Supports role hierarchies and separation of duties

**Examples:** Admin role, Read-Only role, Billing role in enterprise apps

**Pros:**
- Easy to understand and manage for many organizations
- Good fit for environments with stable, role-oriented duties

**Cons:**
- Role explosion in complex organizations (too many roles)
- Lack of fine-grained context (time, location, attributes)

**Use cases:** Enterprise applications, on-prem systems, systems with well-defined job functions.

---

## 25. ABAC (Attribute-Based Access Control) — Access Control model

**Definition:** Authorization model where access decisions are based on attributes of users, objects, actions, and environment (e.g., user.department, resource.sensitivity, timeOfDay).

**Key features:**
- Very flexible and fine-grained
- Policy-based using attributes and rules (e.g., XACML)
- Can evaluate dynamic context (risk score, location)

**Examples:** Access granted if user.department=='finance' AND resource.classification=='internal' AND timeOfDay between 9-17

**Pros:**
- Highly dynamic and precise control
- Good for complex, context-aware policies

**Cons:**
- Complexity in policy authoring and attribute management
- Performance overhead for real-time evaluation

**Use cases:** Cloud environments, dynamic access needs, cross-organization data sharing.

---

## 26. Risk-based authentication (RBA) / Risk Adaptive Authentication

**Definition:** Similar to adaptive auth; uses risk signals to determine required authentication strength. The system assesses risk and can trigger step-up authentication.

**Key features:**
- Uses device reputation, IP risk, behavioral signals, velocity checks
- Step-up flows (MFA) on suspicious activity
- Often backed by ML scoring or rule engines

**Examples:** Step-up when login from unusual country or new device

**Pros:**
- Good balance of security and UX
- Reduces unnecessary friction for legitimate users

**Cons:**
- Requires telemetry, ML models, and tuning to reduce false positives
- Privacy concerns if collecting many signals

**Use cases:** Banking, financial services, high-value transactions, enterprise access.

---

## 27. Capability-based authentication (capability tokens / object-capability model)

**Definition:** Security model where a capability (unforgeable token) is given to holders granting specific rights to an object—possession of the token is sufficient to exercise authority.

**Key features:**
- Capabilities are fine-grained tokens—often bearer-like but scoped to actions/resources
- Emphasizes principle of least authority; tokens can be passed to delegate capabilities
- Often used in capability-based security systems and microkernel OS designs

**Examples:** Signed capability tokens that allow invoking a specific API operation, object-capability languages (e.g., E, capability-based OS concepts)

**Pros:**
- Very granular control over delegated rights
- Fits well with least privilege and distributed systems

**Cons:**
- Token management and revocation can be complex
- If capabilities are bearer tokens, theft leads to misuse unless mitigations applied

**Use cases:** Microservices delegation, distributed object systems, fine-grained third-party integration.

---

## 28. Additional notes & references

- **Combine methods** where appropriate: use RBAC or ABAC for authorization, token/OAuth for API auth, and MFA + risk-based checks for user logins.
- **Defense-in-depth:** validate input, use allowlists, normalize Unicode, and protect secrets via vaults.
- **Standards to review:** OAuth 2.0, OpenID Connect, SAML, FIDO2/WebAuthn, X.509, XACML for ABAC.

---

*Author: Garry Bamrah — 2025*
