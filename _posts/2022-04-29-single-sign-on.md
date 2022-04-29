---
layout: post
title: "Sign Sign-On (SSO)"
summary: "Covering the basics"
author: souzaeduardoac
date: '2022-04-29 17:30:23 -0300'
category: security
thumbnail: /assets/img/posts/sso.png
permalink: /blog/single-sign-on-covering-the-basics/
---

Single Sign-On or SSO is a form of authentication that allow the same user ID to be use across several independent softwares.
It uses a single authentication provided by *System A* that grants authorization to several other systems, passing an authentication token to the requesting application, provide that application is properly configurated in the SSO System.

We are able to see this kind of authentication everywhere, almost every big software accepts a SSO authentication, i.e.g. Spotify allows you to connect with several accounts (Facebook, Apple and Google). Each of this three provides a SSO authentication to the Spotify System and the Spotify is configurated to use each one of them.

SAML, OIDC, LDAP, OAuth2 and Kerberos are all useful for different authorization and authentication purposes.
The protocol you choose should atend to your application needs and what existing infrastructure is in place.

## SSO Protocols

Here we will be looking into this three protocols:

* SAML
* OpenID Connect
* LDAP
* OAuth2
* Kerberos

Every protocol has his uses but all follow the same base workflow

### SAML (Security Assertion Markup Language)
The SAML protocol uses XML files to exchange users, identity provider and applications data, it also provides requests to access secure resources, including authentication and attributes providing user data.
1. The client application requests to access a resource.
2. The application checks with the identity provider if the user is allowed to access it.
3. The identity provider authenticates the user.
4. If the user has access, returns an assertion that the user should be able to access the resource.
5. If the user does not has access, blocks the access.

### OIDC (OpenID Connect)
The OIDC is an authentication and authorization based on the OAuth 2.0 authorization standard, it can also manage API access, using JWT (JSON Web Token) and can integrate with several identity providers.
An interest thing about the OIDC is that it has three main flows:
* `Implicit` – Mainly used for single-page applications only. Tokens are granted directly to the application via a redirect.
* `Authorization Code` – Mainly used for both, mobile and web apps. This flow uses cryptographically-signed JWT tokens and does not share user data with the application.
* `Hybrid` – Combines both flows. Tt returns an ID token to the application via redirect URI. Then, the application submits the ID Token and receives a temporary access token.

1. The user requests access to an application.
2. The application checks with the identity provider.
3. The identity provider authenticates the user. 
4. (If configured) If the user has access, the identity provider displays a message requesting to grant access to the required application, this application may gather the users data from the identity provider.
5. The identity provider generates an Token with users data that the required application.
6. The identity provider redirects the user back to the application, and the user can access it while the application uses the data given by the identity provider.
OpenID Connect Use Cases
OIDC supports both secure authentication and authorization, as well as API access. There are three main flows you can use for different use cases:

### LDAP (Lightweight Directory Access Protocol)
The LDAP is a protocol that provides access to a directory of credentials, which can be shared between multiple applications.
As for your workflow, follow this rule:
1. The client application requests access to data stored in the LDAP database
2. The client application provides the user's access credentials to the LDAP server.
3. The LDAP server verifies that the data entered matches the credentials for that user stored in the database.
4. If the credentials match the stored user ID, LDAP will allow the client application to access the requested information.
5. If the credentials are incorrect, LDAP blocks access.

### OAuth
OAuth 2 is an authorization protocol that enables applications to obtain limited access to user accounts on an HTTP service, such as Facebook, GitHub, and Google.
1. The client application requests authorization for access service resources.
2. If approved, the application receives an authorization grant.
3. The application requests an access token from the authorization server (API). This step requires the identity and the authorization grant given by the provider.
4. If the application identity is authenticated and the authorization grant is valid, the provider issues an access token to the application.
5. The application requests the resource from the API and presents the access token for authentication.
6. If the access token is valid, the API serves the resource to the application.
7. If the access token is invalid, the API returns a 401 Not Authorized response

### Kerberos
Kerberos is a network authentication protocol designed to provide strong authentication for client/server applications.
1. Client requests an authentication ticket from the Key Distribution Center.
2. The Key Distribution Center verifies the credentials and sends back an encrypted authentication ticket and session key.
3. Client requests to access an application on a server. A ticket request for the application server gets sent to the Key Distribution Center which consists of the client’s authentication ticket and an authenticator.
4. The Key Distribution Center returns a ticket and a session key to the user.
5. The ticket is sent to the application server. Once the ticket and authenticator have been received, the server can authenticate the client.
6. The server replies to the client with another authenticator. On receiving this authenticator, the client can authenticate the server.