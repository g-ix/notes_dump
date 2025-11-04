OIDC:

OIDC authentication is an identity layer built on top of the OAuth 2.0 protocol that allows users to sign in to a website or application using their credentials from an identity provider like Google, Microsoft, or Facebook. 
It provides an identity layer to the OAuth 2.0 authorization layer by introducing an ID token, which contains user identity information. OIDC can be used to enable single sign-on (SSO), where a user can sign in once to access multiple applications.

---

How it works
1) A user attempts to access a protected application (the "Relying Party" or RP). 
2) The RP redirects the user to an OIDC provider (like Google or Facebook) to authenticate them. 
3) The OIDC provider verifies the user's identity, often by checking a session cookie or prompting for login. 
4) The user is asked to authorize the RP to access their information. 
5) The OIDC provider redirects the user back to the RP with an authorization code. 
6) The RP uses this code to request an ID token and an access token from the OIDC provider. 
7) The ID token, a secure JSON Web Token (JWT), contains user information (claims) like name and email, which the RP uses to confirm the user's identity and log them in. 

---

Key components
- OpenID Provider (OP): The service that authenticates the user and provides identity information (e.g., Google, Azure AD). 
- Relying Party (RP): The application or website that wants to authenticate the user. 
- ID Token: A JWT containing claims about the authenticated user. It is used for user identification. 
- Access Token: A token used for authorization, allowing the RP to access specific resources on behalf of the user. 
- Authorization Code: A temporary code exchanged for the ID and access tokens. 
- Discovery Document: A JSON file published by the OIDC provider at a specific URL (e.g., https://server.com/.well-known/openid-configuration) that contains metadata about the provider's endpoints and supported features. 

---

Benefits
- Improved user experience: Enables single sign-on (SSO), so users don't have to create and remember separate passwords for every application. 
- Enhanced security: Offloads the responsibility of user identity verification to expert providers, reducing the need for applications to handle and store sensitive user credentials. 
- Standardized authentication: Creates a standard way for applications to authenticate users, making it easier to integrate with different identity providers. 
