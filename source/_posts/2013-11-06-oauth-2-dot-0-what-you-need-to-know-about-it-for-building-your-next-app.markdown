---
layout: post
title: "OAuth 2.0 - What you need to know about it for building your next app"
date: 2013-11-06 22:42
comments: true
categories: OAuth, API, Github
---

OAuth 2.0 is an authorization framework that allows applications to gain access to an HTTP service.  It has become the *de facto* standard for users to share data from one website account with another website.  You may have already used OAuth without even realizing it.  The popular login options "Login with Facebook" and "Login with Twitter" are both utilizing the OAuth authorization process defined in RFC6749 &mdash; namely OAuth 2.0.

When building a web application, OAuth is making it easy for developers to tie in with other websites, though there are some terms that first need to be understood to comprehend exactly how access can be gained and granted.

Authentication is the process in which one user identifies herself based on some shared secret like a username and password or a token.  Authorization, on the other hand, dictates the level of access an authenticated user has to any given resource.  A token or hash (not to be confused with the `Hash` data-type) is merely a generated, hard-to-guess string of characters. 

Token-based authorization such as OAuth has the advantage of eliminating the need for a user to share her password with a 3rd party app, allowing the user to restrict the level of access, and permitting the user to revoke access to her data. 

In order to ensure security without compromising the ease of use, OAuth 2.0 requires that all communication is sent over an SSL connection.

Let's look at a concrete example of how we can use OAuth to gain access to a users GitHub account within our app.

<br/>
### 1. Register the application

First and foremost, we need to establish a trust relationship between our application *MyApp* and Github.  This is done through means of using a shared client ID and client secret which is generated when [registering our app with Github](https://github.com/settings/applications/new).  When using these credentials in API requests, Github can properly identify the authenticity of our app.

<br/>
### 2. Authorization request & grant

```
+-------+                               +------------+
|       |--(1)- Authorization Request ->|   GitHub   |
| MyApp |                               | (Resource  |
|       |<-(2)-- Authorization Grant ---|   Owner)   |
+-------+                               +------------+
```

In order to initiate a request to gain access to certain data in the user's Github account, we craft a special link in our application `MyApp` which, when clicked, will forward the user to GitHub with an *authorization request*.  It looks like this:

```
GET https://github.com/login/oauth/authorize/?client_id=71d3cfc29d&scope=user:email&redirect_uri=https://myapp.com/callback
```

When looking closer at the link, we see three parameters:

* `client_id` - This is the secret shared identifier that Github has previously provided to us when we registered our app.
* `scope` - It describes what resources our app wants access to.  The available scopes are defined by Github.
* `redirect_uri` - Tells Github where to send the *authorization grant* to.


Github will use these parameters to validate the authenticity of the request and find out what resources is requested.  It presents a prompt to the logged in Github user asking whether`MyApp` should be allowed to access a certain set of resources in their account. When the user agrees to grant access, Github will send an HTTP `GET` request to the `redirect_uri` and appends a `code` parameter with contains the authorization grant.

```
GET https://www.myapp.com/?code=4dabfce522efeecaac6819
```


<br/>
### 3. Access Token

```
+-------+                               +----------------+
|       |--(3)-- Authorization Grant -->|     GitHub     |
| MyApp |                               | (Authorization |
|       |<-(4)------ Access Token ------|     Server)    |
+-------+                               +----------------+
```

Now that `MyApp` has the authorization grant, it can contact Github and request an access token, which is used for the actual API calls. To do that `MyApp` sends a `POST` request to Github including the client ID, client secret and the authorization grant code.

```
POST https://github.com/login/oauth/access_token
 client_id=71d3cfc29d
 client_secret=c826145889facbca
 code=4dabfce522efeecaac6819
```

If Github's authorization server receives valid credentials, it will respond with an access token.  This access token allows `MyApp` to send API requests on behalf of the user for resources belonging to the previously defined `scope`.

```
{ access_token: c826145889facbcae488411a45339a19 }
```

<br/>
### 4. Making API calls
```
+-------+                               +-----------+
|       |--(5)----- Access Token ------>|   Github  |
| MyApp |                               | (Resource |
|       |<-(6)--- Protected Resource ---|   Server) |
+-------+                               +-----------+
```

At this point, `MyApp` can make API calls to retrieve information from the Github user account, and it is authorized with the access token.  The access token can be put into either the header:

```
Authorization: token c826145889facbcae488411a45339a19
```

… or alternatively, added as a parameter to the API call URI:

```
https://api.github.com/user?access_token=c826145889facbcae488411a45339a19
```

<br/>
### Additional information

For a detailed breakdown of OAuth 2.0, I can highly recommend to read [RFC6749](http://tools.ietf.org/html/rfc6749), especially section [4](http://tools.ietf.org/html/rfc6749#section-4) and [5](http://tools.ietf.org/html/rfc6749#section-5).



