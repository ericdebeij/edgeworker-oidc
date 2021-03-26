# OpenID connect with OneLogin at the Akamai Edge
This EdgeWorker can be used as a sample or starting point of how to perform IDP integration at the Edge using OpenID Connect to the OneLogin IDP.

To be used as sample code. Lots of topics not implemented yet, amongs others:
- Nonce
- Logout

## Components
- Identity Provider, for example OneLogin
- Application delivered via Akamai
- Origin with protected content, can be static storage like NetStorage or S3
- EdgeWorker to implement authentication logic
- Proxy to Identity Provider

## EdgeWorker resources
The EdgeWorker currently supports the following resources:
### /.../login?url=redirect-url
Initiate the login process.

Parameters:
- url: Used upon successful login to redirect to the original requested url. Default /
   - Special value: debug_block - the callback will not be performed (give the developer to do the callback locally using the sandbox)
   - Special value: debug_info - instead of redirect detailed information about the response of the callback will be provided.
### /.../callback?code=authorization-code
Used upon succesful authentication at the IDP to request the JWT tokens, create the akamai_token and perform the redirect to the origal requested url.

# Configuration
For the sake of clearity the following example endpoints are used in the documentation and configuration files
1. Application domain
   - Example: application.example.com
   - The EdgeWorker is available via /oidc/
1. OIDCsupport internal domain
   - Example: oidcsupport.example.com
   - The IDP is proxied via /onelogin/
1. IDP domain
   - Example: example.onelogin.com

## IDP provider
The application needs to be configured at the IDP provider. In general the following is a result of the configuration at the IDP provider:
1. IDP domain
   - Example: https://example.onelogin.com
   - Authentication URL - https://example.onelogin.com/oidc/2/auth
   - Token URL - https: https://example.onelogin.com/oidc/2/token
1. IDP application
   - ClientID (OIDC_CLIENTID)
   - Client secret (OIDC_SECRET)
   - Login URL - https://application.example.com/oidc/login
   - Callback URL - https://application.example.com/oidc/callback

## EdgeWorker
Multiple endpoints are defined in the EdgeWorker and needs to be changed in the responseProvider
1. oidcContext.auth
   - Authentication URL - https://example.onelogin.com/oidc/2/auth
1. oidcContext.token
   - Token URL proxied via oidcsupport - https://oidcsupport.example.com/onelogin/oidc/2/token
1. oidcContext.service
   - Service URL at oidcsupprt for services to be delegated to property manager - https://oidcsupport.example.com/service

## Property Manager - application
The application involves the EdgeWorker only during login time. When a valid and not-expired token is available access will be granted without involvement of the EdgeWorker.
### Variables
User defined variables are required in order to share the required credentials with the EdgeWorker.
1. OIDC_AKSECRET - The key used to generate the token
1. OIDC_CLIENTID - The client id of the OpenID provider
1. OIDC_SECRET - The secret to be used in combination with the client id 
### Rules
At the application level their needs to be two rules:
1. An unprotected path to involve the EdgeWorker
   - IF path matches /oidc/*
      - Edgeworker
1. Protected area's are protected using an Akamai token (using the key as specified in OIDC_AKSECRET)
   - IF NOT path matches /oidc/*
      - Validate token
      - IF NOT valid token
         - Redirect /oidc/login?url=urlEncode(URL)
   
## Property Manager - oidcsupport
A second property manager configuration is required to support the EdgeWorker. This configuration can be shared between applications. The rules for this configuration are in oidcsupport_rules.json. As there is an advanced behavior in there, it has to be setup by Professional Services. 

For every IDP an endpoint needs to be configured with its own origin (Forward host header:origin) and the path prefix should be removed.
- IF path matches /onelogin/*
   - origin: example.onelogin.com (forward host header = origin host header)
   - remove path prefix







