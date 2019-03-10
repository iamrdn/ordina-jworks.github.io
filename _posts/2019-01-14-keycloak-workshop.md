---
layout: post
authors: [johan_silkens, jeroen_meys]
title: 'Keycloak workshop'
image: /img/keycloak/logo.png
tags: [Spring,Angular,Keycloak]
category: Workshop
comments: true
---

# Table of contents

1. [Intro](#intro)
2. [Setting up our project](#setting-up-our-project)
3. [Setting up Keycloak](#keycloak)
4. [Front-End setup](#front-end)
5. [Back-End setup](#back-end)


# Intro

Everyone who has written software as a novice or professional has to deal with user authentication and authorization at some point. Either implemented it themselves or handle the requests via an Identity Management Platform or IDP for short. Today, you can find several platforms that handles user login requests. Like Keycloak, OKTA, OpenAM , ... etc. All those platforms have their own features and possibilities that may be useful for your use case. 

In this blog post, we will focus on [Keycloak](https://www.keycloak.org/). An open-source identity and access management platform (IAM) from Red Hat's [Jboss](http://www.jboss.org/). We have chosen for Keycloak because of it's well-comprehensive documentation and it's availability of connectors to choose form. Of course there's a lot you can configure with Keycloak and it's supported libraries for numerous programming languages and frameworks. We will cover just the basics to get you started.

Keycloak comes with several handy features build-in like:

- Two-factor authentication
- Bruteforce detection
- Social login (Facebook, Twitter , Google , … )
- LDAP/AD integration 
- … And much more



In the following example, we will implement security in a forum application. When you clone and run the project, you will notice that you can do everything on the forum. Even when you're not authenticated. 

# setting up our project

Keycloak supports [multiple ways](https://www.keycloak.org/docs/latest/server_installation/index.html#_operating-mode) to be set up.
For non-production purposes, it's easiest to just [download](https://www.keycloak.org/downloads.html) and run the standalone.
In our case, the latest release version is `4.8.3.Final`.

```bash
curl https://downloads.jboss.org/keycloak/4.8.3.Final/keycloak-4.8.3.Final.zip --output keycloak-4.8.3.zip
unzip keycloak-4.8.3.zip
cd keycloak-4.8.3.Final/bin/
./standalone
```

## Keycloak

## Front-end

Let's begin with an initial project to start with. Check out the repository and do the regular `npm` commands to install the necessary dependencies and run the project 
```bash
 $ git clone https://github.com/lenn3k/ngx-broken-blog
 $ npm install
 $ npm run start
```

You can now point your browser to `http://localhost:4200` With no security implemented. Everyone can post or edit forum posts in our application.

There are several libraries that could work with KeyCloak. It is wise to choose a library that is actively maintained and well-implemented. The OpenID Foundation has a [list](https://openid.net/certification/) of certified OpenID libraries.

We will use Manfred Steyers's [Angular-oauth2-oidc](https://github.com/manfredsteyer/angular-oauth2-oidc) library for our front-end.

In the front-end, import the Angular-oauth2-oidc into our project with:

`$ npm install angular-oauth2-oidc`

After the installation is finished, let's implement the `OauthModule` into `App.module.ts` and `App.components.ts` respectively to begin with.

**App.module.ts**

```typescript
...
import {OAuthModule} from 'angular-oauth2-oidc';
...

imports: [
    ...
   OAuthModule.forRoot({
      resourceServer: {
        allowedUrls: [environment.backEndUrl],
        sendAccessToken: false
      }
    })  
    ...
]
```

**App.components.ts**

```typescript
...
constructor(private oauthService: OAuthService) {
    this.oauthService.redirectUri = window.location.origin;
    this.oauthService.clientId = environment.keycloak.clientId;
    this.oauthService.scope = 'openid profile email';
    this.oauthService.issuer = environment.keycloak.url;
    this.oauthService.tokenValidationHandler = new JwksValidationHandler();
...
```

Next create a config file where we specify our KeyCloak realm and clientId

**Auth-password.config.ts**

```typescript
import {AuthConfig} from 'angular-oauth2-oidc';

export const authPasswordFlowConfig: AuthConfig = {
  issuer: 'http://localhost:9080/auth/realms/brokenblog', // the realm endpoint to navigate to
  redirectUri: window.location.origin + '/', //url to go to after login
  clientId: 'web_app',
  scope: 'openid profile email',
  // disable in prod env
  showDebugInformation: true,
  oidc: false
};
```



## Back-end

Next 

## 



