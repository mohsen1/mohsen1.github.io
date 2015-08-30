---
layout: post
title:  "OAuth 2.0 and JWT in Swagger"
date:   2015-07-25 19:48:19
tags: [swagger]
---

To define an API that supports [OAuth 2.0](https://tools.ietf.org/html/rfc6749) and uses [JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519) for security in Swagger we need to understand OAuth and JWT first.

### OAuth 2.0
Traditionally if an API owner wants to secure their API they use a mean of authentication like [HTTP Basic Authentication](http://tools.ietf.org/html/rfc2617) to protect their API. If they decide to share the resources behind that API with some other developer they would have to share their credentials with them. Sharing your password with others is never a good idea. OAuth is fixing this issue by introducing an authorization mechanism which doesn't involve sharing main credentials from the API owner.

API servers that use OAuth authenticate requests using an access token instead of credentials. The access token can be obtained by the client in various ways. Access token can have limited scope of access to different parts of the API or it can have a limited lifetime.
