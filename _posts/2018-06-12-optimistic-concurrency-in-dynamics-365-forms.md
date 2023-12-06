---
title: Optimistic Concurrency in Dynamics 365 Forms
date: 2018-06-12s00:00:00 +0300
categories: [Dynamics 365]
tags: [dynamics365]     # TAG names should always be lowercase
img_path: /assets/img/posts/2018-06-12-optimistic-concurrency-in-dynamics-365-forms
---

Imagine this scenario: two users access the same Contact record at the same time, they both update the emailaddress1 field with different values (personal email/business email) and finally, they save the changes. As a result of this, the first update will be overridden by the second one and if the Audit setting is not enabled, the data will be gone forever. This is a text-book example of one of the problems of not implementing a concurrency control mechanism, and a possible one in Dynamics 365 with changes happening in parallel by multiple users.

# Optimistic Concurrency

Since CRM 2015 Update 1, it’s possible to use Optimistic Concurrency control to avoid potential data loss. The idea is simple, the entities now have a property called RowVersion (auto incremental number) which stores the retrieved version of the record. When an Update or Delete request is executed (and the ConcurrencyBehavior has been explicitly configured), Dynamics 365 will compare if the record RowVersion is the same than the one stored in the database and rollback the operation if they’re not.

The [MSDN](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/org-service/reduce-potential-data-loss-using-optimistic-concurrency) page has good examples to understand how the functionality works. The feature has some limitations and one of those is that it can’t be used in forms using OOB functionality.

> Optimistic concurrency behavior can only be set through an SDK call. There is presently no setting for it in a form of the Web application.

# Optimistic Concurrency in forms — Attempt 1

Microsoft doesn’t provide a setting to enable this in forms so when I first approached this requirement I thought it was achievable using a plugin. The plan was to create a simple plugin in the Update message (pre-operation) and just set the ConcurrencyBehavior to force the check. Of course, I was wrong. The problem is that the RowVersion property is never passed to the plugin target entity and therefore there’s no way of comparing them.

![Null rowversion](1-null.png)
_Null rowversion_

It’s possible to read more about this problem in the [Community Forum](https://community.dynamics.com/crm/f/117/t/220118).

# Optimistic Concurrency in forms — Attempt 2

This was a setback, but I was still sure that the approach would work. Dynamics 365 form is retrieving the record, so it must have the RowVersion property stored somewhere, it was just matter of finding where and pass it manually to the plugin (as a workaround for the bug). It took me a while, but I was able to discover a JS function to get it:

`formContext.data.entity.getRowVersion();`

*Disclaimer: I haven’t found any official or unofficial documentation about this function, the MSDN doesn’t mention it so there’s no way for me to know if it’s supported or not.*

With this SDK client API, a custom field and a JS function on the OnSave event on the form, I was able to pass the RowVersion to the plugin but again, I run into some issues when trying to set the RowVersion and ConcurrencyBehavior properties in the plugin:

`request.Target.RowVersion = request.Target.GetAttributeValue<string>("fjo_rowversion"); 
request.ConcurrencyBehavior = ConcurrencyBehavior.IfRowVersionMatches;`

If these two properties are set on the PreOperation stage, Dynamics is going to completely ignore them and no validation will take place. If we apply the changes during the PreValidation stage, we’ll face a strange error which makes me think that the RowVersion property is being ignored:

> Microsoft.Crm.CrmException: The RowVersion property must be provided when the value of ConcurrencyBehavior is IfVersionMatches.

To work around this problem while Microsoft fix it, we can implement our custom check to validate if the RowVersion of the record is different than the one stored in the database and throw an InvalidPluginExecutionException if it’s the case. After doing it, we finally have our optimistic concurrency control in place:

![Working example](2-example.png)
_Example_

You can check the complete plugin implementation (Dynamics 365 solutions included) in [Github](https://github.com/fedejousset/OptimisticConcurrency).

# Conclusions

Optimistic Concurrency control is a really good feature but it’s not there for forms. Even though it’s possible to implement it as we saw in this post, it’s still has a couple of drawbacks:

1. A field and some client side customizations are required (Max Ewing suggested to use the PreImage RowVersion property but it’s also null) which means that we’ll have to edit every entity where we want to apply the check (getting the version from the Target object would allow us to create a generic solution with no changes at entity level)
2. OOB Concurrency control can’t be used in the plugin forcing us to implement a custom one which will also require some performance tests in order to guarantee that this solution is Production ready

I really hope we’ll have a fix for these problems in a future release and hopefully I’ll be able to write another post about it.