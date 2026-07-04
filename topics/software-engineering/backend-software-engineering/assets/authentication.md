# Authentication

#deep-dive

API authentication is the process of verifying the identity of clients attempting to access an API, ensuring that only authorized users or applications can interact with the API’s resources.

---
## Token Authentication

Token-based authentication is a protocol which allows users to verify their identity, and in return receive a unique access token. During the life of the token, users then access the website or app that the token has been issued for, rather than having to re-enter credentials each time they go back to the same webpage, app, or any resource protected with that same token.

*JWT Tokens* are the most common implementation of Token Authentication.

---
## JWT Authentication

`JWT` (JSON Web Token) is an open standard for securely transmitting information between parties as a JSON object. It consists of three parts:

- a header (which specifies the token type and algorithm used for signing)
- a payload (which contains the claims or the data being transmitted)
- a signature (which is used to verify the token’s integrity and authenticity).

JWTs are commonly used for authentication and authorization purposes, allowing users to securely transmit and validate their identity and permissions across web applications and APIs. They are compact, self-contained, and can be easily transmitted in HTTP headers, making them popular for modern web and mobile applications.

A common way to use JWTs is OAuth Bearer Tokens.

### JWT structure example

![JWT Structure](jwt-decoded.png)

### How JWT works

1. User signs up, password is hashed and stored.
2. User signs in, password is hashed and compared against the stored hash.
3. After successful authentication, the server issues a JWT with appropriate claims, signed with the secret/private key.
4. Front-end stores the JWT and includes it in requests using the Bearer token format.
5. The server verifies the JWT using the public key (for RS256) or shared secret (for HS256) to validate the token and its authenticity before processing the request.

---
## Basic Authentication

Basic Authentication is a simple HTTP authentication scheme built into the HTTP protocol. It works by sending a user’s credentials (username and password) encoded in base64 format within the HTTP header.

---
## Cookie or Session Based Authentication

Cookie-based authentication is a method of maintaining user sessions in web applications.

When a user logs in, the server creates a session and sends a unique identifier (session ID) to the client as a cookie. This cookie is then sent with every subsequent request, allowing the server to identify and authenticate the user.

### Cookie for Session Based Auth example

```mathematica
Set-Cookie: sessionId=abc123xyz456; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600`
```

---
## Open Authentication (OAuth)

OAuth is an open standard for authorization that allows third-party applications to access a user’s resources without exposing their credentials.

It works by issuing access tokens after users grant permission, which applications then use to interact with resource servers on behalf of the user. This process involves a resource owner (the user), a resource server (which holds the data), and an authorization server (which issues tokens).

OAuth enables secure, token-based access management, commonly used for granting applications permissions to interact with services like social media accounts or cloud storage.

### How OAuth works and why it exists.

Developers build a lot of APIs. The API Economy is a common buzzword you might hear in boardrooms today. Companies need to protect their REST APIs in a way that allows many devices to access them. In the old days, you’d enter your username/password directory and the app would login directly as you. This gave rise to the delegated authorization problem.

“How can I allow an app to access my data without necessarily giving it my password?”

If you’ve ever seen one of the dialogs below, that’s what we’re talking about. This is an application asking if it can access data on your behalf.

![OAuth example](oauth.png)

This is OAuth.

OAuth is a delegated authorization framework for REST/APIs. It enables apps to obtain limited access (scopes) to a user’s data without giving away a user’s password. It decouples authentication from authorization and supports multiple use cases addressing different device capabilities. It supports server-to-server apps, browser-based apps, mobile/native apps, and consoles/TVs.

You can think of this like hotel key cards, but for apps. If you have a hotel key card, you can get access to your room. How do you get a hotel key card? You have to do an authentication process at the front desk to get it. After authenticating and obtaining the key card, you can access resources across the hotel.

To break it down simply, OAuth is where:

1. App requests authorization from User
2. User authorizes App and delivers proof
3. App presents proof of authorization to server to get a Token
4. Token is restricted to only access what the User authorized for the specific App

### OAuth tokens

- Access token
  The token that the client uses to access the Resource Server (API). They're short lived - think in hours and minutes.
- Refresh token
  This is much longer-lived; days, months, years. This can be used to get new tokens. To get a refresh token, applications typically require confidential clients with authentication. Refresh tokens can be revoked.
  When revoking an application’s access in a dashboard, you’re killing its refresh token. This gives you the ability to force the clients to rotate secrets. What you’re doing is you’re using your refresh token to get new access tokens and the access tokens are going over the wire to hit all the API resources. Each time you refresh your access token you get a new cryptographically signed token. Key rotation is built into the system.

#### Token formats

The _OAuth_ spec doesn’t define what a token is. It can be in whatever format you want. Usually though, you want these tokens to be _JSON Web Tokens_ (a standard). In a nutshell, a `JWT` (pronounced “jot”) is a secure and trustworthy standard for token authentication. JWTs allow you to digitally sign information (referred to as claims) with a signature and can be verified at a later time with a secret signing key.

