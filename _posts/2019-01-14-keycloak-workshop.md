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

Everyone who has written software as a novice or professional has to deal with user authentication and authorisation at some point. Either implemented it themselves or handle the requests via an Identity Management Platform or IDP for short. 

Today, you can find several platforms that handles user login requests. Like Keycloak, OKTA, OpenAM , ... etc. All those platforms have their own features and possibilities that may be useful for your usecase. 

In this blog post, we will focus on [Keycloak](https://www.keycloak.org/). An open-source identity and access management platform (IAM) from Red Hat's [Jboss](http://www.jboss.org/). We have chosen for Keycloak because of it's well-compreshensive documentation and it's availability of connectors to choose form.

Of course there's a lot you can configure with Keycloak and it's supported libraries for numerous programming languages and frameworks. We will cover just the basics to get you started.

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

Let's begin with an initial project to start with. Check out 
```bash
 $ git clone https://github.com/lenn3k/ngx-broken-blog
 $ npm install
 $ ng serve
```


 With no security implemented. Now everyone can for example post or edit forum posts in our application.



In the front-end, import the <oauth library> into our project with:

`$ npm install <keycloak lib`

## Back-end

Next 

## 



