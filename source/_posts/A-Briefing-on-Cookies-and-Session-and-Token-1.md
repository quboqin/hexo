---
title: 'A Briefing on Cookies, and Session and Token'
date: 2019-07-11 16:29:26
tags:
---


HTTP or HTTPs is a stateless protocol. So when we want to exchange information between different pages or different sites under the same domain, we need Coookies to 'memory' information on the previous pages. 

<img src="/A-Briefing-on-Cookies-and-Session-and-Token-1/cookies.png" width="70%" padding="5px">

1. The client send the first request to the server
2. The server then respones this request and Set-Cookie into the client
3. In the follow requests, the client will send requests with this cookie

We want to keep secret information on the server side, then we create a seesion on the server side, and only send a session id to the client side. 

But this approach will cause another problem how to share these information between servers. 

If we keep all information in the Cookies, it is not safe. If we keep the information in Sessions, the scalability on the server side is a new problem.

Here is Token, or JWT(Json Web Token). There are two main differences between Token and the previous methods.
1. First we still keep the serect information on the client side, but we add a time stamp and encrypt this information, then signature this token. So on the client we can't modify this information. 

2. Though we can save the token into Cookies, but it is still unsafe. So we keep these token into the local storage. So it will not automatically be read and sent to the server(This will cause the CSRF attack). We need to read it and attach it into the Authorization field in HTTP header mannually.

3. And because we keep these information on the client, we have no scalabilty problem on the server side



