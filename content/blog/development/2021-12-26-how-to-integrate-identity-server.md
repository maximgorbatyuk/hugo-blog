---
layout: post
title: How to integrate Identity Server into your web application
category: development
tags: [.net, opinion, tutorial]
date: "2021-12-26"
Description: "There is a tutorial how to integrate and use IdentityServer4 in your web application"
---

Let’s imagine you have a web application built as monolith and you want to introduce microservices. Or you may have several clients connected to your backend solution: mobile app, SPA, devices, etc. One of the first tasks that you have to solve is integrating authentication and authorization. In my opinion, one of the simplest way is integrating SSO (Single-Sign-On system) into your application.

There is an opensource ready-to-use product [IdentityServer4](https://github.com/IdentityServer/IdentityServer4) which implements [OpenID Connect](https://openid.net/connect/) and [OAuth2.0](https://datatracker.ietf.org/doc/html/rfc6749) frameworks. Is is built using .NET core 3.1 and easy to modify according to your business rules. Also, it is a out-of-box solution ready to deploy. Therefore, you don’t have to develop and setup a custom authentication system.

## Versions of IdentityServer
There are two versions of the IS application: free to use opensource IS4 and [commercial IdentityServer5](https://duendesoftware.com/products/identityserver). According to documentation, IS5 is free for development and testing, but you should pay for using it on the production. IS4 is declared as legacy system, but it is free to use on the production.

Even though all new features are developing in the commercial IS5, you may start to go live with free IS4. The IS4 system contains all staff required by Open Id connect and OAuth2.0 frameworks.

## How to integrate the IS4
### Downloading the IS4 solution from the GitHub
To integrate the IS4 into your system, you just need to download it from [samples](https://github.com/IdentityServer/IdentityServer4/tree/main/samples/Quickstarts). I’d suggest you to choose [my extended solution](https://github.com/maximgorbatyuk/IdentityServerForAll) but you still may choose one of [the original ones](https://github.com/IdentityServer/IdentityServer4/tree/main/samples/Quickstarts).

My version contains the following:

1. The IS4 solution without any storage. You are free to integrate your favorite one.
2. Several samples of clients including [OAuth 2.0 debugger](https://oauthdebugger.com)
3. Custom profile service where you can write your code related to issuing claims

Feel free to consider my repository as an instruction to integrate the IS4 from the original repository.

### Setup your IS4

1. Add your own Scope to restrict access to different APIs (like [here](https://github.com/maximgorbatyuk/IdentityServerForAll/blob/master/src/IdentityServer/Config/IdentityConfig.cs#L26)). If your application has now domain segregation with different scopes, you may not use the custom scope or just use a single one. Here I use “core.api” as a key of the scope, but you may choose any other name.
2. Add clients of the IS4 ([like here](https://github.com/maximgorbatyuk/IdentityServerForAll/blob/master/src/IdentityServer/Config/IdentityConfig.cs#L30)). To proof the concept, I am adding a web-browser-debug client like [this](https://github.com/maximgorbatyuk/IdentityServerForAll/blob/master/src/IdentityServer/Config/Clients/WebBrowserStubClient.cs#L7). The client allows me to see claims which is being encrypted in the JWT token. Also, don’t forget to mention your own scope in the clients’ allowed scopes property (like [here](https://github.com/maximgorbatyuk/IdentityServerForAll/blob/master/src/IdentityServer/Config/Clients/InteractiveMvcClient.cs#L26)).
3. __Optional__ Add external login providers like Google authentication if it necessary. [Here](https://github.com/maximgorbatyuk/IdentityServerForAll/blob/master/src/IdentityServer/Config/ExternalAuthProviders.cs#L13) I have a sample code which integrates the Google. Also, the Facebook, GitHub, ActiveDirectory, etc, providers are available to be used.
4. __Optional__ In [this Custom profile service](https://github.com/maximgorbatyuk/IdentityServerForAll/blob/master/src/IdentityServer/Config/CustomProfileService.cs#L20) you may change claims which will be used to prepare a JWT token for clients.

### Setup you Web API application

Here I will give you example using ASP.NET core Web Api. I believe it is easy to find tutorials of integrating OAuth2.0 authentication services for other web frameworks for other programming languages.

1. Add Bearer authentication with the IS4 url address (like [this](https://github.com/maximgorbatyuk/IdentityServerForAll/blob/master/src/WebApi/Startup.cs#L16)).
2. __Optional__ Add scope authorization to restrict accesses (like [this](https://github.com/maximgorbatyuk/IdentityServerForAll/blob/master/src/WebApi/Startup.cs#L30)).

If you do the step 2, and your Client without the scope does a web-request, it will get 403 error.

### Setup debug client like [OAuth 2.0 debugger](https://oauthdebugger.com)

1. Add the client like [this](https://github.com/maximgorbatyuk/IdentityServerForAll/blob/master/src/IdentityServer/Config/Clients/WebBrowserStubClient.cs#L14).
2. Go to url below:

```
https://localhost:6001/connect/authorize?response_type=id_token&client_id=client&client_secret=secret&redirect_uri=https%3A%2F%2Foauthdebugger.com%2Fdebug&scope=openid%20email%20profile&nonce=wnpup8t4v2b
```
