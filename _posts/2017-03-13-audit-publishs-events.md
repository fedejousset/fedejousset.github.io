---
title: Audit Publish & PublishAll events
date: 2017-03-13 00:00:00 +0300
categories: [Dynamics 365]
tags: [dynamics365, plugin, audit]     # TAG names should always be lowercase
img_path: /assets/img/posts/2017-03-13-audit-publishs-events
---

Based on a question that I found in [Stack Overflow](http://stackoverflow.com/questions/39265795/crm-plugin-for-publish-and-publish-all-messages/39291051) I did a little test to see how the Publish and PublishAll events work and what information is available to audit these events using a plugin.

# Requests involved

The first step was to find out what are the requests involved and what properties exist to log the changes that are being published. After some research in MSDN, it's pretty clear that there are two requests involved: [PublishXmlRequest](https://msdn.microsoft.com/en-us/library/microsoft.crm.sdk.messages.publishxmlrequest.aspx) and [PublishAllXmlRequest](https://msdn.microsoft.com/en-us/library/microsoft.crm.sdk.messages.publishallxmlrequest.aspx).

In the first request, it's possible to get a list of all the components being published using the [ParameterXml](https://msdn.microsoft.com/en-us/library/microsoft.crm.sdk.messages.publishxmlrequest.parameterxml.aspx) property, but in the PublishAll event this property is not there so it's not possible to know which components are involved (actually we know that all resources are being published but there's no detail).


# Plugin

With this in mind, I wrote a really simple plugin to log these events:
https://gist.github.com/fedejousset/e2695bcf10cfb4b339f1d53d4ac8289c

Once the plugin is registered in CRM, it's necessary to create the following steps:

![PublishAll](step_publishall.png)
_PublishAll_

![Publish](step_publish.png)
_Publish_

# Testing plugin

As you can see in the code, when the plugin is executed it will create a record of a custom log entity that I've created:

![Publish Log](publish_log.png)
_Publish Log_

As usual, you can check and download the code (and the CRM solution) from [GitHub](https://github.com/fedejousset/PublishAudit).