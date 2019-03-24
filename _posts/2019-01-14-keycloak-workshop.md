---
layout: post
authors: [jeroen_meys, johan_silkens]
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

In this blog post, we will focus on [Keycloak](https://www.keycloak.org/). An open-source identity and access management platform (IAM) from Red Hat's [Jboss](http://www.jboss.org/). We have chosen for Keycloak because of it's well-comprehensive documentation and it's availability of connectors to choose form. Of course there's a lot you can configure with Keycloak and it's supported libraries for numerous programming languages and frameworks. We will cover just the basics just to get you started.

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
$ curl https://downloads.jboss.org/keycloak/4.8.3.Final/keycloak-4.8.3.Final.zip --output keycloak-4.8.3.zip
$ unzip keycloak-4.8.3.zip
$ cd keycloak-4.8.3.Final/bin/
$ ./standalone
```

## Keycloak

## Front-end

Let's begin with an initial project to start with. Check out the repository and do the regular `npm` commands to install the necessary dependencies and run the project 
```bash
 $ git clone https://github.com/lenn3k/ngx-broken-blog
 $ npm install
 $ npm run start
```

You can now point your browser to `http://localhost:4200` .The initial project has no security implemented so everyone can post or edit forum posts in our application.

There are several libraries that could work with KeyCloak. It is wise to choose a library that is actively maintained and well-implemented. OpenID Foundation has a [list](https://openid.net/certification/) of certified OpenID libraries. In this project, We will use Manfred Steyers's [Angular-oauth2-oidc](https://github.com/manfredsteyer/angular-oauth2-oidc) library for our front-end.

In the front-end, import the Angular-oauth2-oidc into our project with: `$ npm install angular-oauth2-oidc` After the installation is finished, let's implement the `OAuthModule` into `App.module.ts` and `App.components.ts` respectively to begin with.

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

Next create a config file where we specify our KeyCloak realm and ClientId

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



With the provided library, it's possible to either redirect to KeyCloak's login page, or handle our login request directly. Which is possible via `authPasswordFlowConfig`

**Login-page.component.ts** (in constructor)

```typescript
this.oAuthService.configure(authPasswordFlowConfig);
this.oAuthService.loadDiscoveryDocument();
```

The last statement will fetch the Discovery document from KeyCloak. A Discovery document is a JSON document that contains the configuration of the OAuth provider. Including URI's of the authorization, token , userinfo and public-key endpoints. It can be retrieved from `http://localhost:9080/auth/realms/brokenblog/.well-known/openid-configuration` .

After a successful login, we want to go back to the home screen

**Login-page.component.ts**

```typescript
...

this.oAuthService.fetchTokenUsingPasswordFlowAndLoadUserProfile(username, password)
        .then(value => {
          this.router.navigate(['/']);
        })
...        
```



## Back-end

- checkout Spring boot project

```bash
$git clone git@github.com:ordina-jworks/keycloak-excercise-blog-server.git blog
$cd blog
$./gradlew bootRun
```

The blog server is now running without any security.  
Our frontend application already sends a token with every request, but Spring boot ignores this for now.  
We will now see how we can use Keycloak to verify the authenticity of the token and authorize the caller to access resources.  

## keycloak-spring-boot-starter

We Will now see how to secure a Spring Boot 2.1 application by verifying the JWT token and authorizing access to resources.
When only using the Spring framework, Keycloak has an adapter that can be used, but luckily, Spring Boot is all about "Opinionated Defaults Configuration".
Just like many other starter dependencies, Keycloak provides its own: `keycloak-spring-boot-starter`.  

There used to be a `keycloak-spring-boot-2-starter`, but this has been renamed once more to `keycloak-spring-boot-starter` for better alignment with the Spring Boot eco system.  

 All we have to do is add it to the classpath and provide some extra application specific configuration.
 The import:  
 
```gradle
dependencies {
    ...
    implementation 'org.keycloak:keycloak-spring-boot-starter:4.8.3.Final'
}
```

 And tell the starter where it can find our Keycloak server and what settings to use in application.properties:

```properties
keycloak.realm=brokenblog
keycloak.auth-server-url=http://localhost:9080/auth/
keycloak.resource=web_app
keycloak.public-client=true
keycloak.bearer-only=false
```

The final step is to define some endpoint authorization settings.  
Let's say we want only users (and admins) to be able to modify content:  

```properties
keycloak.security-constraints[0].authRoles[0]=user
keycloak.security-constraints[0].authRoles[1]=admin
keycloak.security-constraints[0].securityCollections[0].methods[0]=POST
keycloak.security-constraints[0].securityCollections[0].methods[1]=PUT
keycloak.security-constraints[0].securityCollections[0].methods[2]=DELETE
keycloak.security-constraints[0].securityCollections[0].patterns[0]=/api/v1/*
```

If we run the application now, we can verify if our defined security configuration is working:

```shell
$# GET something from /api/v1/ -> should have no security
$curl -i http://localhost:8080/api/v1/topics
HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Tue, 19 Mar 2019 13:29:51 GMT

[]
$
$# POST something to /api/v1/ -> should be for users only
$curl -i -X POST http://localhost:8080/api/v1/topics
HTTP/1.1 302 
Set-Cookie: JSESSIONID=5088C063048B9AF25CC7229981F0D4A8; Path=/; HttpOnly
Set-Cookie: OAuth_Token_Request_State=2ac750f4-4a87-4883-af3e-1517a7ac50b6; Version=1; HttpOnly
Location: http://localhost:9080/auth/realms/brokenblog/protocol/openid-connect/auth?response_type=code&client_id=web_app&redirect_uri=http%3A%2F%2Flocalhost%3A8080%2Fapi%2Fv1%2Ftopics&state=2ac750f4-4a87-4883-af3e-1517a7ac50b6&login=true&scope=openid
Content-Length: 0
Date: Tue, 19 Mar 2019 13:31:55 GMT
```

Notice how we don't get a HTTP 401 (Unauthorized) or a HTTP 403 (Forbidden) but a HTTP 302 (Redirect) which redirects the caller to the login form.   
We can use the Keycloak API to obtain a token and try again. We will store the token in an environment variable `TOKEN`.  

```shell
$export TOKEN=$(curl -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=web_app" \
  -d "username=jeroen" \
  -d "password=jeroen" \
  -d "grant_type=password" \
  -X POST http://localhost:9080/auth/realms/brokenblog/protocol/openid-connect/token | jq -r .access_token)
$echo $TOKEN
eyJhbGciOeUNvcWVJVWxNIn0.eyJqdGkiOiI......Hnz5aFdcAiB-5o-yep6rcGP_H6yQoW
```

This makes it easy to obtain a token without using an actual login form.
Let's post again using our token:

```shell
$# POST something to /api/v1/ -> should be for users only
$curl -i -X POST -H "Authorization: bearer $TOKEN" http://localhost:8080/api/v1/topics
HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 20 Mar 2019 13:43:50 GMT

[]
```

This is the most basic form of integrating Keycloak in Spring Boot.  
However, you most likely want to use [Spring Security](https://spring.io/projects/spring-security) for a more fine-grained authorisation mechanism.  
Luckily, the Keycloak starter is perfectly compatible with Spring Security!


## Keycloak and Spring Security 5
  
All we need to do is add the dependency to our classpath and tweak some settings.

```gradle
dependencies {
    ...
    implementation 'org.springframework.boot:spring-boot-starter-security'
}
```

The version of Spring Security is chosen by the `io.spring.dependency-management` plugin.  
Because we used `2.1.3.RELEASE` as our Spring Boot version, Spring Security `5.1.4.RELEASE` was chosen.  

We will configure our Spring Boot application so that it uses our token data (claim) for authentication instead of a session.  
This means our application is stateless and our token will be re-validated with every request.    
Note that we lose some functionality compared to a session based mechanism this way: We don't have a server sided session in which we can store data (like a shopping basket).  
This can also be regarded as an advantage: We don't have to provide dedicated session storage for every logged-in user!  

Let's see how to configure our application with Spring Security added to our dependencies.
First we do some clean up in our application.properties file:

```properties
keycloak.realm=brokenblog
keycloak.auth-server-url=http://localhost:9080/auth/
keycloak.resource=web_app
keycloak.public-client=true
keycloak.bearer-only=true
keycloak.principal-attribute=preferred_username
```

```java
import org.keycloak.adapters.KeycloakConfigResolver;
import org.keycloak.adapters.springboot.KeycloakSpringBootConfigResolver;
import org.keycloak.adapters.springsecurity.KeycloakConfiguration;
import org.keycloak.adapters.springsecurity.config.KeycloakWebSecurityConfigurerAdapter;
import org.keycloak.adapters.springsecurity.management.HttpSessionManager;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.session.SessionRegistryImpl;
import org.springframework.security.web.authentication.session.NullAuthenticatedSessionStrategy;
import org.springframework.security.web.authentication.session.RegisterSessionAuthenticationStrategy;
import org.springframework.security.web.authentication.session.SessionAuthenticationStrategy;

@KeycloakConfiguration
public class SecurityConfiguration extends KeycloakWebSecurityConfigurerAdapter {

    // use application.properties instead of WEB-INF/keycloak.json
    @Bean
    public KeycloakConfigResolver KeycloakConfigResolver() {
        return new KeycloakSpringBootConfigResolver();
    }

    // no sessions needed: we go bearer-only
    @Override
    protected SessionAuthenticationStrategy sessionAuthenticationStrategy() {
        return new NullAuthenticatedSessionStrategy();
    }

    // use Keycloak as authentication provider
    @Override
    protected void configure(AuthenticationManagerBuilder auth) {
        auth.authenticationProvider(keycloakAuthenticationProvider());
    }

    // Since Spring Boot 2.1, bean overwriting is not allowed by default and we would get:
    // he bean 'httpSessionManager', defined in class path resource [be/ordina/blog/config/SecurityConfiguration.class], could not be registered. 
    // A bean with that name has already been defined in URL ....
    // 
    // Making the HttpSessionManager conditional is a clean way to work around this issue.
    //
    // A more dirty way would be to allow bean overwriting via:
    // spring.main.allow-bean-definition-overriding=true
    @Bean
    @Override
    @ConditionalOnMissingBean(HttpSessionManager.class)
    protected HttpSessionManager httpSessionManager() {
        return new HttpSessionManager();
    }

    // define our endpoint security.
    // we disable CSRF for now since this is out of scope for our tutorial.
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
        //@formatter:off
        http
            .csrf()
                .disable()
            .authorizeRequests()
                .antMatchers("/api/v1/topics").authenticated()
                .antMatchers("/api/v1/**").hasAuthority("admin")
                .and()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        //@formatter:on
    }
}
```

The last part of our HttpSecurity configuration `.sessionCreationPolicy(SessionCreationPolicy.STATELESS)` is important here:  
without it, Spring Security would still set up a session for us - even though we indicated in our Keycloak config that we want bearer-only mode.  

We can verify if our setup is working after a restart:

```shell
$curl -i http://localhost:8080/api/v1/topics
HTTP/1.1 401 
WWW-Authenticate: Bearer realm="Unknown"
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 21 Mar 2019 21:00:20 GMT

{
    "timestamp":1553202020870,
    "status":401,
    "error":"Unauthorized",
    "message":"Unauthorized",
    "path":"/api/v1/topics"
}
```

Notice how we don't get redirected (HTTP 302) like before but immediately get a 401. This is because we opted for bearer-only.  
Let's add our bearer token again:

```shell
$curl -i -X GET -H "Authorization: bearer $TOKEN" http://localhost:8080/api/v1/topics
HTTP/1.1 401 
WWW-Authenticate: Bearer realm="brokenblog", error="invalid_token", error_description="Token is not active"
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
WWW-Authenticate: Bearer realm="brokenblog", error="invalid_token", error_description="Token is not active"
Content-Length: 0
Date: Thu, 21 Mar 2019 20:09:52 GMT
``` 

Oh no, our token is no longer valid! At least we now know that Keycloak checks the expiry date in the claim.  
Readers who pay close attention might have also spotted that the `WWW-Authenticate` is sent back twice.  
At the moment of writing, I still have to figure out why this is happening. All I know for now is that it shouldn't!
Anyway, after obtaining a new token, we try again:

```shell
$curl -i -H "Authorization: bearer $TOKEN" http://localhost:8080/api/v1/topics
HTTP/1.1 200 
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 21 Mar 2019 21:28:51 GMT

[]
```

We can now use all the added benefits of Spring Security!

# Conclusion

## 



