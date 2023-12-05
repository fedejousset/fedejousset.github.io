---
title: Dynamics 365 Web API Postman Collection
date: 2018-09-17 00:00:00 +0300
categories: [Dynamics 365]
tags: [dynamics365, postman]     # TAG names should always be lowercase
img_path: img/2018-09-17-dynamics-365-web-api-postman-collection/
---

I started to use Postman probably four years ago and it immediately became the only reason why this Firefox fanboy began to use Chrome (it was first developed as a Chrome App). A lot has changed with Postman since those old days, but it’s still is extremely useful while developing or testing API’s.

I was pleased when Microsoft recently added the Use Postman with Web API section to the documentation but it’s not widely known so I’ve created a custom Postman collection (environment and collection with sample requests) to simplify the process.

# Prerequisites

To use this collection, you need to download and install Postman and also register an Azure Active Directory Application to connect and authenticate user accounts. If you haven’t done this before, follow this walk-through making sure that Implicit Authentication is enabled (this can be done updating the Application manifest once created):

![Update manifest](1-manifest.png)
_Updating manifest to allow OAuth Implicit Flow_