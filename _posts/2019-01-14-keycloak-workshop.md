---
layout: post
authors: [johan_silkens, jeroen_meys]
title: 'Keycloak workshop'
image: /img/keycloak/logo.png
tags: [Spring,Angular,Keycloak]
category: Workshop
comments: true
---

//Table of contents

# Intro

Everyone who writes software (for a living) has implemented a way to identify and authenticate users. Either in the application itself or via an Identity management platform or IDP for short. 

Today, you can find several platforms that handle user login requests. Like Keycloak, OKTA, OpenAM , ... etc. All those platforms have their own features and possibilities that may be useful for your usecase. 

In this blog post, we'll focus on [Keycloak](https://www.keycloak.org/). An open-source identity and access management platform (IAM) from Red Hat's [Jboss](http://www.jboss.org/). We have chosen for Keycloak because of it's well-compreshensive documentation and a lot of connectors to choose form.

//Key features of keycloak

...

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

Let's begin with an initial project to start with. Check out `<url>`
 With no security implemented. Now everyone can for example post or edit forum posts in our application.



In the front-end, import the <oauth library> into our project with:

`$ npm install <keycloak lib`

## Back-end

Next 

## 



