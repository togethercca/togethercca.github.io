---
permalink: "/endCustomerApi/"
layout: page
title: "End Customer Api"
---

Overview
========

This document describes the resources that make up the end customer API.

* TOC
{:toc}

Current Version  
===============  
You can find the current Version of the API specification as swagger file and a browsable version of it here:   

* [swagger file](https://ccaonline-dev.ccaedv.at/endCustomerApi/swagger.json)  
* [browsable version](https://ccaonline-dev.ccaedv.at/swagger-ui/)  

Security Considerations
=======================

The EndCustomerApi exposes confident data of endcustomers to the internet.
We encourage you to expose and use the API, even for development ant test purposes, only over HTTPS.
Please make sure that the web-server serving the API is secured and has the latest security patches installed.

Schema
======

All data is sent and received as JSON.

Blank fields are included as `null` instead of being ommited.

All timestamps are returned in ISO 8601 format:
`YYYY-MM-DDTHH:MM:SSZ`

Parameters
==========

Many API methods take optional parameters. For GET requests, any parameters not specified as a segment in the path can be passed as an HTTP query string parameter:

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons/42/claims?perPage=50&page=4"
```

In this example the '42' values are provided for the `{personId}` path parameters while `perPage` and `page` are passed in the query string.

For POST, PUT, DELETE requests, parameters not included in the URL should be encoded as JSON with a Content-Type of 'application/json'.

```powershell 
Invoke-RestMethod -Uri "https://eca.ccaedv.at/api/token" -Method Post -Body (ConvertTo-Json @{ grant_type= "password"; username= "alan"; password= "complete"}) 
```

Client Errors
=============

* Querying an entity or sublists for an entity that does not exist will result in a `404 Not Found`.
* Missing path parameters will result in `404 Not Found`.
* Invalid syntax or invalid types in requests will result in `400 Bad Request` responses.
* Requests that cannot be processed will generate a `422 Unprocessable Entity` response.

### Corellation Id

Most responses will contain a X-Correlation-Id Header that can be used to lookup an error message in the ccaonline logfiles.

```powershell
HTTP/1.1 400 Bad Request
Content-Length: 11
X-Correlation-Id: 0000000-0000-0000-1e78-0080000000fc
```

### Removed API actions

Using an api call that is not supported anymore will return a `410 Gone` response.

```powershell
HTTP/1.1 410 Gone
```

HTTP Verbs
==========

|Verb    | Description									|
|--------|:---------------------------------------------|
| HEAD   | Retrieve just the HTTP header info			|
| GET    | Used for retrieving resources.				|
| POST   | Used for creating resources.					|
| PUT    | Used for replacing resources or collections.	|
| DELETE | Used for deleting resources.					|


Authentication
==============

The EndCustomerAPI supports the OAuth2 Bearer Authentication.
Requests that require authentiation will return `403 Forbidden` if not authentication is provided.

## OAuth2 Bearer Token in Header

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons/42" -Headers @{Authorization="TOKEN"}
```

Pagination
==========

Requests that return multiple items will be paginated to 15 items by default. You can request further pages with the `page` parameter. You can also request a larger page size (up to 100) by providing the `perPage` parameter.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?page=4&perPage=100"
```
If the page number is not provided the first page will be returned.
Page numbering starts with 1.

If a page is requested that is out of range the result will contain an empty data list.

## Pagination Result

The pagination result contains the following properties:

| Name       | Description                                                                                  |
| ---------- | -------------------------------------------------------------------------------------------- |
| data       | resource data as array                                                                       |
| pageNumber | current page's number (1-based)                                                              |
| pageSize   | number of items returned in a page                                                           |
| first      | url to the first page of results                                                             |
| last       | url to the last page of results                                                              |
| next       | url to the immediate next page of results. (empty if the current page is the last page)      |
| prev       | url to the immediate previous page of results. (empty if the current page is the first page) |
| totalCount | total count of items available                                                               |


## Ordering

List results can be sorted using `orderBy` parameters. 
The parameter takes a string in the form of `<propertyName> "-" asc/desc` where `<propertyName>` must be the name of a property that is part of the model returned by the list action.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?orderBy=fullName-desc"
```
Returns a list of persons orderd by the `fullName` - property in descending order.

### Order by multiple properties

You can order by mulitple properties by providing the orderBy parameter multiple times.
The parameters will be resolved from left to right.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?orderBy=fullName-asc&orderBy=birthday-desc"
```

Returns a list of persons sorted by `fullName` (in ascending order) and `birthday` (in descending order).

Data Filtering 
==============

Filtering on the list api actions can be achieved by providing the property names as query parameter key and the filter expression as the associated value. The property names are case-insensitive.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?birthday=2016-05-19"
```

If an value is specified that does not match the property type, the api responds with `400 Bad Request`. 
Date values must be passed in the form `YYYY-MM-DD`.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?birthday=2016-05-19T10:25:00.928Z

HTTP/1.1 400 Bad Request
```

This query gets persons with birthdays after the 1. Jan 1990

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?birthday=2016-05-19
```

## Filter Operations

The following filter operations are available:

| Filter Operation | Description  |
| ---------------- | ------------ |
| sw               | starts with  |
| cn               | contains     |
| eq               | equal to     |
| gt               | greater than |
| lt               | less than    |

The default filter operation, that will be applied if none is specified, is based on the type of the value.

|Datatype| Default Filter Operation|
|--------|:------------------------|
| string | sw                      |
| date   | eq                      |
| number | eq                      |


However, the operation can be overridden with a second query parameter, with a key of the structure `<propertyName>+op`and the operation as value. The available alternative operations are also based on the property type. The filter operation values are case insensitive and will be normalized to lowercase.

|Datatype| Available Filter Operations| 
|--------|----------------------------|
| string | sw, cn, eq, gt, lt         |
| date   | eq, gt, lt                 |
| number | eq, gt, lt                 |


This query gets persons with birthdays after the 1. Jan 1990

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?birthday=1990-01-01&birthday+op=gt"
```

If an operation is specified that is not available in general, the api responds with `400 Bad Request`.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?birthday=1990-01-01&birthday+op=gaussian"

HTTP/1.1 400 Bad Request
```

If an operation is specified, but not the respective filter expression, the api responds with `400 Bad Request`.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?birthday+op=eq"

HTTP/1.1 400 Bad Request
```

If an operation is specified that is not available for the type, the api responds with (400) Bad Request.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?birthday=2016-05-19&birthday+op=sw"

HTTP/1.1 400 Bad Request
```

The identifier property cannot be part of a filter request, because gt and lt range queries are not supported and an exact-match-query can be accomplished with the respective details request. If it is specified the api responds with (400) Bad Request.
Example
```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?id=1000

HTTP/1.1 400 Bad Request
Content-Length: 11
X-Correlation-Id: 00000000-0000-0000-1c78-0080000000fc
```

### Multiple Filters

If multiple filters are passed they are always logically composed with AND.

This query gets people born on the 1. Jan 1990 named steve.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?birthday=1990-01-01&fullName=steve"
```

### Case Sensitivity

String filters always work case-sensitively.

The /persons resource contains persons with fullnames "Steve Buscemi" and "stephen colbert"

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?fullName=Ste"

HTTP/1.1 200 Ok

{
	...
	fullName: "Steve Buscemi",
	...
}
```

### Filter Ranges

To perform a range query with two specified bounds, the same property name is passed twice in the query parameters, with two different values. The parameters are position-sensitive, with the leftmost value being the lower bound and the rightmost value being the upper bound. The bounds are always interpreted as inclusive.

This query gets persons with a fullname beginning with A, everyone in between until the last one beginning with B.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?fullName=A&fullName=B"
``` 

For range queries, strings are matched with startsWith by default, and dates and numbers by their exact values. These operations cannot be overridden. If an operation is specified on a property that is part of a range query, the api responds with `400 Bad Request`.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?fullName=A&fullName=B&fullName-op=eq"

HTTP/1.1 400 Bad Request
```

If a property is specified more than twice, the api responds with (400) Bad Request.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?fullName=A&fullName=B&fullName=C"

HTTP/1.1 400 Bad Request
```

Two equal bound expressions are allowed, to ease bound computation on the client-side. In this case, the query will degenerate to a single-expression-query with the default range query operation applied.


This query gets persons with a full name starting with A.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?fullName=A&fullName=A"
```

This query gets persons with an ANP equal to a thousand

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?ANP=1000&ANP=1000"
```

Boolean values cannot be part of a range query.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons?isCompany=false&isCompany=true"

HTTP/1.1 400 Bad Request
```

Multi-Language Support 
======================

If the Accept-Language Header is set, the api will return dimensional data translated to the requested language, if a translation exists.

```powershell
Invoke-RestMethod -Uri "https://eca.ccaedv.at/endcustomer/api/v0.1/persons/42" -Headers @{'Accept-Language'='en-us'}
```
