---
title: Dynamics 365 Web API Postman Collection
date: 2018-09-17 00:00:00 +0300
categories: [Dynamics 365]
tags: [dynamics365, postman]     # TAG names should always be lowercase
img_path: /assets/img/posts/2018-09-17-dynamics-365-web-api-postman-collection
---

I started to use Postman probably four years ago and it immediately became the only reason why this Firefox fanboy began to use Chrome (it was first developed as a Chrome App). A lot has changed with Postman since those old days, but it’s still is extremely useful while developing or testing API’s.

I was pleased when Microsoft recently added the Use Postman with Web API section to the documentation but it’s not widely known so I’ve created a custom Postman collection (environment and collection with sample requests) to simplify the process.

# Prerequisites

To use this collection, you need to download and install Postman and also register an Azure Active Directory Application to connect and authenticate user accounts. If you haven’t done this before, follow this walk-through making sure that Implicit Authentication is enabled (this can be done updating the Application manifest once created):

![Update manifest](1-manifest.png)
_Updating manifest to allow OAuth Implicit Flow_

# Importing/Configuring D365 Web API Collection

The first step is to [download and import](https://github.com/fedejousset/Dynamics365WebApiPostmanCollection "download and import") the Dynamics 365 Web API collection using the Run in Postman button from its [GitHub repository](https://github.com/fedejousset/Dynamics365WebApiPostmanCollection "GitHub repository"):

![Run in Postman](2-download.png)
_Use the Run In Postman button to download the collection_

This button will open Postman and import the “Dynamics 365 CE Web API” collection and the “Dynamics 365 Environment (Online)”:

![Import collection](3-import.png)
_Collection and Environment imported_

Once imported, you need to configure the environment to connect to your Dynamics 365 instance updating the required [environment variables](https://www.postman.com/docs/v6/postman/environments_and_globals/variables "environment variables"):

 - **ApplicationId**: Azure Application ID used to authenticate the user (variable format: 9f0e560f-4f2e-4a56-afb8-af72f9f20d06)
 - **RedirectUrl**: Redirect URL configured in the Azure AD Application (variable format: https://localhost)
 - **Resource**: Dynamics 365 Organization that has been configured in the AAD Application Delegated Permissions section (variable format: https://yourorg.crm4.dynamics.com)

 To update these values, please follow the steps described in the image:

![Update variables](4-variables.png)
_Update environment_

# Using the D365 Web API Collection

The collection contains 40+ template requests to allow you to quickly compose and test your own requests. These templates are split in the following six folders:

- Basic Operations (Create, Update, Delete)
- Retrieve Operations (Retrieve, Retrieve Multiple, Change Tracking)
- Batch Operations
- Functions (Bound/unbound functions with/without parameters)
- Actions (Bound/unbound actions with/without parameters)
- Metadata (Retrieve entity/attribute metadata)

Before executing any of these requests, you need to authenticate yourself against the Dynamics 365 instance:

1.  Click on the ellipses (…) next to the collection and select "Edit":

![Update variables](5-edit.png)
_Update environment_

2. Select the Authorization tab
3. Click the Get New Access Token button
4. Complete authentication settings and click the Request Token button. Note that environment variables (showed in orange in the image) are being used here meaning you can use the same collection to connect to multiple environments:

- **Grant Type:** Implicit
- **Callback URL:** {{RedirectUrl}}
- **Auth URL:** https://login.microsoftonline.com/common/oauth2/authorize?resource={{Resource}}
- **Client ID:** {{ApplicationId}}

![Authentication settings](6-auth.png)
_Authentication settings_

5. Postman will show the Microsoft login page. Enter your credentials.
6. The access token and the expiration time (in minutes) will be displayed. Scroll down and click the Use Token button:

![Token](7-token.png)
_Token_

7. Click the Update button to close the Authentication window and save the token

Once the token has been saved, all the template requests will inherit it from the collection. At this point, you can proceed to execute one of the sample requests to verify that everything is working by selecting it and clicking the Send button and review the response:

![Example response](8-example.png)
_Response_

I know it feels like a lot of steps, but once you get familiar with the authentication process you will be able to do it in just a few seconds:

![Full authentication process](8-full.gif)
_Full authentication process_

After getting a successful response you’re ready to start playing around with the sample requests or create your owns. Please remember that the access token will expire after one hour (unless configured otherwise in Azure AD) and you will need to get a new one following the previous steps.

The samples in the collection only cover the most common cases I’ve faced over the last couple of weeks so if you want to include new ones, I’ll be waiting for your Pull Requests in [GitHub ](https://github.com/fedejousset/Dynamics365WebApiPostmanCollection "GitHub").

On a final note, collections don’t automatically sync when they are updated so if you found this collection useful, you should check periodically the [GitHub repository](https://github.com/fedejousset/Dynamics365WebApiPostmanCollection "GitHub repository") for new versions and download/import it as explained before.