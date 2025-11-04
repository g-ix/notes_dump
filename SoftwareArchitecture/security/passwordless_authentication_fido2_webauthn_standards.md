# 🔐 Authentication Standards: FIDO2, WebAuthn, Passkeys & Biometrics

## 🧭 Overview

Modern authentication is moving **beyond passwords** — toward **biometric** and **cryptographic** methods that are more secure, privacy-preserving, and user-friendly.

This document covers:

1. Basics → Advanced concepts of **FIDO2**, **WebAuthn**, **Passkeys**, and **Biometric Authentication**
2. How they actually work under the hood
3. Implementation examples (frontend + backend)
4. How these technologies interact

---

## 🪪 1. The Problem with Passwords

- Passwords can be **stolen**, **phished**, or **reused**.
- 81% of data breaches are caused by weak or stolen passwords.
- Traditional **2FA** adds friction (OTP, SMS, etc.) and can still be hijacked.

**Goal:** Eliminate passwords while maintaining or improving security.

---

## 🧱 2. The Foundation — FIDO Alliance

**FIDO** = *Fast IDentity Online*

Founded by Google, Microsoft, Apple, and others, FIDO sets standards for **passwordless authentication** using **public-key cryptography**.

### 🔹 FIDO2 consists of:
1. **WebAuthn (Web Authentication API)** — browser API that lets web apps perform public-key authentication.
2. **CTAP2 (Client to Authenticator Protocol 2)** — communication protocol between devices (like your phone or security key) and the client (browser).

---

## ⚙️ 3. WebAuthn: The Core API

### 🔸 What it does
- Enables **public key credential creation** and **assertion** directly from a browser.
- Replaces passwords with **cryptographic key pairs** stored securely in a device.

### 🔸 Basic Flow
1. **Registration**
   - User registers a credential (key pair) with the service.
   - The device generates:
     - A **public key** → sent to server
     - A **private key** → stored securely (e.g., TPM, Secure Enclave)
   - The server stores only the **public key**.

2. **Authentication**
   - The server sends a **challenge**.
   - The device signs the challenge using the **private key**.
   - The server verifies the signature with the **public key**.

---

## 🔑 4. Passkeys: The User-Friendly Layer

**Passkeys** are a **user-friendly implementation of FIDO2/WebAuthn**.

They:
- Store the cryptographic keys **in the device** (or cloud keychain).
- Allow syncing across devices (iCloud, Google Password Manager, etc.).
- Replace traditional passwords entirely.

✅ Example:
- Instead of typing a password, you use **Face ID**, **Touch ID**, or **PIN** to unlock your passkey.

---

## 🧬 5. Biometric Authentication

Biometrics = Using your **body features** as identity proof.

### Common Modalities
| Biometric Type | Example | Security Source |
|----------------|----------|------------------|
| Fingerprint | Touch ID / Android fingerprint | Secure Enclave / Trusted Execution |
| Face | Face ID, UAEICP scan | Neural Face Mapping |
| Iris | Samsung iris unlock | Infrared eye mapping |

### 🔹 Important:
Biometric data **never leaves your device**.  
It only **unlocks** the private key — your fingerprint or face isn’t sent to the website.

---

## 🧠 6. How It All Works (Step-by-Step)

### 🪩 Registration (Creating a Passkey)
1. User visits your site → clicks “Register with Passkey”.
2. Server generates a **challenge** (random string).
3. Browser calls:
   ```js
   navigator.credentials.create({
     publicKey: { challenge, rp, user, pubKeyCredParams }
   })
   ```
4. The authenticator (device) creates a key pair.
5. Public key → sent to your server.
6. Private key → stays securely in device.

### 🪩 Authentication (Login)

1. Server sends a challenge to browser.
2. Browser calls:
```js
navigator.credentials.get({
  publicKey: { challenge, allowCredentials }
})
```
3. Device asks user for biometric (e.g., Face ID).
4. Private key signs the challenge.
5. Server verifies the signature using stored public key.
6. ✅ User authenticated — no password needed.

## ⚒️ 7. Implementation Example 

🧩 Frontend (WebAuthn JS)
```js
// Registration example
async function registerPasskey() {
  const options = await fetch("/register-options").then(res => res.json());
  const credential = await navigator.credentials.create({ publicKey: options });
  await fetch("/register", {
    method: "POST",
    body: JSON.stringify(credential)
  });
}

// Authentication example
async function loginPasskey() {
  const options = await fetch("/login-options").then(res => res.json());
  const assertion = await navigator.credentials.get({ publicKey: options });
  await fetch("/login", {
    method: "POST",
    body: JSON.stringify(assertion)
  });
}
```

🧩 Backend (Node.js + @simplewebauthn/server)
```js
import { generateRegistrationOptions, verifyRegistrationResponse } from '@simplewebauthn/server';

// Generate registration challenge
app.get('/register-options', (req, res) => {
  const options = generateRegistrationOptions({
    rpName: "MyApp",
    userID: "user123",
    userName: "test@example.com",
  });
  req.session.challenge = options.challenge;
  res.json(options);
});

// Verify registration response
app.post('/register', async (req, res) => {
  const verification = await verifyRegistrationResponse({
    response: req.body,
    expectedChallenge: req.session.challenge,
    expectedOrigin: "https://myapp.com",
    expectedRPID: "myapp.com",
  });

  if (verification.verified) {
    // Save public key and credential ID
    saveCredential(verification.registrationInfo);
  }

  res.json({ success: verification.verified });
});
```

## 🛡️ 8. Security Advantages

| Feature             | Password      | FIDO2 / Passkey  |
| ------------------- | ------------- | ---------------- |
| Phishing resistance | ❌ No          | ✅ Yes            |
| Stored locally      | ❌ Server-side | ✅ Device-secured |
| Replay attacks      | ❌ Possible    | ✅ Impossible     |
| Usability           | 🟡 Depends    | ✅ Seamless       |

## 🌐 9. Supported Platforms
| Platform           | Support |
| ------------------ | ------- |
| Google Chrome      | ✅       |
| Safari (iOS/macOS) | ✅       |
| Firefox            | ✅       |
| Edge               | ✅       |
| Android            | ✅       |
| iOS                | ✅       |

## 🧩 10. Advanced Topics

🔸 CTAP2 (Client to Authenticator Protocol)
Defines how external authenticators (USB keys, NFC devices) communicate with browsers.
Example: YubiKey, Titan Security Key.

🔸 Cross-Device Authentication
Allows you to approve login on your phone even if the browser is on your PC.
Uses Bluetooth + cryptographic channels.

🔸 Enterprise FIDO2
Integration with SSO providers like Okta, Azure AD, or Google Workspace.
Enables passwordless login across corporate environments.

## 🧠 11. Quick Summary
| Concept            | Description                                             |
| ------------------ | ------------------------------------------------------- |
| **FIDO2**          | Security standard for passwordless auth (FIDO Alliance) |
| **WebAuthn**       | Browser API for FIDO2                                   |
| **CTAP2**          | Communication protocol between client and authenticator |
| **Passkeys**       | User-friendly implementation of WebAuthn                |
| **Biometric Auth** | Unlocks device-stored credentials securely              |
| **SSO**            | Centralized login system (enterprise use)               |


🚀 12. Getting Started Checklist

1.Understand how WebAuthn works.
2. Use libraries like:
- @simplewebauthn/server
- webauthn-json

3. Add frontend support:
```bash
navigator.credentials.create() / navigator.credentials.get()
```

4. Store
- credentialID
- publicKey
- signCount

5. Deploy over HTTPS (WebAuthn requires secure origin).
6. Test on Chrome, Safari, and mobile devices.

## 📚 13. Further Reading
[FIDO2 Overview (fidoalliance.org)](https://fidoalliance.org/fido2/)
[WebAuthn Spec (W3C)](https://www.w3.org/TR/webauthn-2/)
[Passkeys.dev (Google)](https://passkeys.dev/)
[SimpleWebAuthn Docs](https://simplewebauthn.dev/)
[Apple: Introducing Passkeys](https://developer.apple.com/passkeys/)

## 💡 TL;DR

> Passkeys = Biometric + Cryptographic + Passwordless Security

Built on FIDO2 + WebAuthn, they make authentication fast, secure, and phishing-proof — the future of digital identity.
