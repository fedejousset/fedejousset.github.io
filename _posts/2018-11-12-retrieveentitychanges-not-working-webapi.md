---
title: RetrieveEntityChanges not working in WebAPI
date: 2017-03-09 00:00:00 +0300
categories: [Dynamics 365]
tags: [dynamics365, webapi]     # TAG names should always be lowercase
---

During the process of migrating an integration from using the Organization Service to the Web API, it has become obvious that the [RetrieveEntityChanges function](https://msdn.microsoft.com/en-gb/library/mt491170.aspx) is not working properly in this endpoint.

I've tested it in all WebAPI available versions, and even though v9 is the only one returning an error it's not working as expected in v8 either.

Let's take the following request and execute it in both version to see the different responses:

`GET /api/data/vX.X/RetrieveEntityChanges(EntityName='account',PageInfo=@PageInfo,Columns=@ColumnsSet)?@PageInfo={'Count':5000,'PageNumber':1,'ReturnTotalRecordCount':true}&amp;@ColumnsSet={'Columns':['accountid']}`

# v8 Response

`{
"@odata.context":"https://xxx.crm11.dynamics.com/api/data/v8.2/$metadata#Microsoft.Dynamics.CRM.RetrieveEntityChangesResponse",
"EntityChanges":{
"MoreRecords":false,
"PagingCookie":null,
"DataToken":"1160042!01/25/2018 14:11:53",
"Changes":[{},{}]
}
}`

In the previous response we can see how the `Changes` collection has the appropriate number of updated items but it's not displaying any field/properties. I tried using several combinations of `ColumnsSet` request object but the result is always the same: no details about the changed records are returned. Microsoft has recognized this as a bug but they've also informed us that the Product team has not prioritized it to be fixed in v8 because there's a workaround in place (see **Workaround...** section).

# v9 Response

`{
"error":{
"code":"",
"message":"<span style="color: #ff0000;"><strong>Resource not found for the segment 'RetrieveEntityChanges'</strong></span>.",
"innererror":{
"message":"Resource not found for the segment 'RetrieveEntityChanges'.",
"type":"Microsoft.OData.Core.UriParser.ODataUnrecognizedPathException",
"stacktrace":"   at Microsoft.OData.Core.UriParser.Parsers.ODataPathParser.CreateFirstSegment(String segmentText)\r\n   at Microsoft.OData.Core.UriParser.Parsers.ODataPathParser.ParsePath(ICollection'1 segments)\r\n   at Microsoft.OData.Core.UriParser.Parsers.ODataPathFactory.BindPath(ICollection'1 segments, ODataUriParserConfiguration configuration)\r\n   at Microsoft.OData.Core.UriParser.ODataUriParser.Initialize()\r\n   at Microsoft.OData.Core.UriParser.ODataUriParser.ParsePath()\r\n   at System.Web.OData.Routing.DefaultODataPathHandler.Parse(IEdmModel model, String serviceRoot, String odataPath, ODataUriResolverSetttings resolverSettings, Boolean enableUriTemplateParsing)\r\n   at System.Web.OData.Routing.DefaultODataPathHandler.Parse(IEdmModel model, String serviceRoot, String odataPath)\r\n   at Microsoft.Crm.Extensibility.OData.CrmODataPathHandler.Parse(IEdmModel model, String serviceRoot, String odataPath)"
}
}
}`

In this case, an [HTTP 404 error code](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.5) is returned which makes me think that the function hasn't been implemented in v9. Microsoft has acknowledge it as a bug too but the same answer was provided, there won't be any fix in the near future because it's possible to retrieve this information in a different way (see next section).

# Workaround (or brand-new way of doing it?)

As it was stated before, Microsoft has recognized these errors but they're not going to fix them because it's possible to keep using the change tracking functionality adding the preference header `odata.track-changes</code>. This header and its implementation is defined as part of the [HTTP 404 error code](https://docs.oasis-open.org/odata/odata/v4.0/cs01/part1-protocol/odata-v4.0-cs01-part1-protocol.html#_Toc365046305), and in my opinion it will replace the RetrieveEntityChanges function (which will be possible deprecated in the future).

Let's convert our previous request in the new format:

`GET /api/data/v9.0/accounts?$select=accountid HTTP/1.1
Prefer: odata.track-changes
Cache-Control: no-cache
OData-Version: 4.0
Content-Type: application/json`

Very straightforward request as you can see, the only difference from a "retrieve multiple" request is the previously mentioned `Prefer` header.

`{
"@odata.context":"/data/v9.0/$metadata#accounts(accountid)/$delta",
"@odata.deltaLink":"/api/data/v9.0/accounts?$select=accountid&amp;$deltatoken=231099!04%2F08%2F2018%2008%3A21%3A20",
"value":
[
{
"@odata.etag":"W/\"915244\"",
"accountid":"60c4e274-0d87-e711-80e5-00155db19e6d"
},
{
"@odata.context":"/api/data/v9.0/$metadata#accounts/$deletedEntity",
"id":"2e451703-c686-e711-80e5-00155db19e6d",
"reason":"deleted"
}
]
}`

If we inspect the `value` property in the previous response, we can see that two records were changed in the system: one of them was created or updated, the other one was deleted. The other interesting property returned is `@odata.deltaLink`, which is the delta link (`DataVersion` property in Organization Service [RetrieveEntityChangesRequest](https://docs.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.retrieveentitychangesrequest?view=dynamics-general-ce-9) we'll have to use to retrieve all the subsequent changes using a request very similar to the first one:

`GET /api/data/v9.0/accounts?$select=accountid&>$deltatoken=231099!04%2F08%2F2018%2008%3A21%3A20 HTTP/1.1
Prefer: odata.track-changes
Cache-Control: no-cache
OData-Version: 4.0
Content-Type: application/json`

You can check out more and better examples in [Microsoft Docs website](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/use-change-tracking-synchronize-data-external-systems.