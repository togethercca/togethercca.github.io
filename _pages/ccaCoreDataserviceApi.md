---
permalink: "/ccaCoreDataservice/"
layout: page
title: "CCA Core Dataservice API"
---

# Overview

This document describes the resources that make up the cca core dataservice api.

* TOC
{:toc}

# Current Version  

You can find the current Version of the API specification as swagger file and a browsable version of it here:   
[specification](https://app.swaggerhub.com/apis/TIS-CCA/cca-core_data_service_api)

# Security Considerations

The CCA Core Dataservice API exposes confident data of endcustomers to the internet.
We encourage you to expose and use the API, even for development and test purposes, only over HTTPS.
Please make sure that the web-server serving the API is secured and has the latest security patches installed.

# Schema

All data is sent and received as JSON.

Blank fields are included as `null` instead of being ommited.

All timestamps are returned in ISO 8601 format:
`YYYY-MM-DDTHH:MM:SSZ`

## OMDS - References
All Entity-Models are crated with OMDS compatibility in mind.
The structure and relationship between entities is closely related to that of the omds 2 specification.
All Properties that have a corrlelating OMDS property are marked in the specification with `x-omds2-property` and the OMDS 2 property name.

```json
 KFZCommonProperties:
    properties:
      besitzerId:
        type: integer
        format: int32
        description: Id of the Person owning the vehicle
        x-cca-property-hint: Person.perId
      fahrzeugArtCode:
        type: string
        x-omds2-property: FzgArtCd
        x-cca-property-hint: kfzArtID
        enum:
          - "999"
          - "ANH"
          - "KRA"
          - "LKW"
          - "MOP"
          - "OMN"
          - "PKW"
          - "PRO"
          - "SON"
          - "ZUG"
```

## CCA - Extensions
All Properties that are specific to cca are marked with `x-cca-extension: true`.
Most properties have a hint to which cca database column is used in the `x-cca-property-hint` extension.


```json
  KFZViewModel:
    allOf:
      - $ref: '#/definitions/KFZCommonProperties'
      - type: object
        properties:
          steuerbefreit:
            type: string
            x-cca-property-hint: KFZSteuerbefreit_T.kfzRisikentbText
            x-cca-extension: true
```

# Parameters

Many API methods take optional parameters. For GET requests, any parameters not specified as a segment in the path can be passed as an HTTP query string parameter:

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen/42/vertraege?perPage=50&page=4"
```

In this example the '42' values are provided for the `{personId}` path parameters while `perPage` and `page` are passed in the query string.

For POST, PUT, DELETE requests, parameters not included in the URL must be encoded as JSON with a Content-Type of 'application/json'.

```powershell 
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/token" -Method Post -Body (ConvertTo-Json @{ grant_type= "password"; username= "alan"; password= "complete"}) 
```

# Client Errors

* Querying an entity or sublists for an entity that does not exist will result in a `404 Not Found`.
* Missing path parameters will result in `404 Not Found`.
* Invalid syntax or invalid types in requests will result in `400 Bad Request` responses.
* Requests that cannot be processed will generate a `422 Unprocessable Entity` response.
* Operations that the user is not authorized to perform will generate a `403 Forbidden` response.

## Corellation Id

Most responses will contain a X-Correlation-Id Header that can be used to lookup an error message in the ccaonline logfiles.

```powershell
HTTP/1.1 400 Bad Request
Content-Length: 11
X-Correlation-Id: 0000000-0000-0000-1e78-0080000000fc
```

## Removed API actions

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


# Prerequisites 

## CCAOnline

The api is delivered as part of the [CCAOnline - Product](https://togethercca.com/produktwelten-cca#cca-online).
A current ccaonline instance is needed to access the api.

## CCA User

The api is always accessed in the context of a cca user.

Operations are authorized based on that users cca - rights.

If a user is not permitted to see / create / edit a cca - entity, the api will return a `403` - status code.

## Registerd Api - Client

To enable the core-data-services the `consuming client` must be registered in ccaonline.

This is done by adding an oauth `client` configuration element to the `sites.config` of the ccaonline installation.
This is done by adding an element to the `<CCAOnlineSettings> <oauth>` element.

The element has the following properties:

- `name`: a friendly display name for the application accessing the api.
- `type`: currently all clients are considered `public` clients.
- `id`: the `client_id` part of the clients credentials.
- `secret`: the `client_secret` part of the clients credentials.

Currently the token endpoint issues JWTs. To create the JWT parameters pointing to the CCAOnline Instance must be configured. Add a `<jwt>` element to the `<CCAOnlineSettings>` with the following properties:

- `audience`: URL of the CCAOnline Instance
- `issuer`: URL of the CCAOnline Instance

Note: Core dataservice api is part of CCAOnline. CCAOnline issues and consumes the token, therefore both is set to the CCAOnline Instance URL.

~~~XML
<?xml version="1.0"?>
<CCAOnlineSettings ...>
  <siteBindings>
    <add name="Default" ...>
    ...
    </add>
  </siteBindings>
  <oauth>
    <add name="client_friendly_name" type="public" id="09647281976743fd09486b462b44facdbcd1a2adfacfb9af6fe81727d7a1005c" secret="2336c8ccc30e5e789c9db62d0167a68f6a5e8c659bf4173b94cee9e1895b9653" />
  </oauth>
  <jwt audience="https://ccds.ccaedv.at/CCAOnline" issuer="https://ccds.ccaedv.at/CCAOnline" />
</CCAOnlineSettings>
~~~


# Authentication

The Core Dataservices supports the OAuth2 Bearer Authentication.
Requests that require authentiation will return `401 not authorized` if not authentication is provided.

## Client Authentication

Client authentication information must be presented when requesting an authentication token.

CCAOnline requires public clients to be registered with the CCAOnline installation.
The registration is done by the sites CCAOnline administrator and encompasses the configuration of client credentials (client_id and client_secret).

[See the RFC 6749](https://tools.ietf.org/html/rfc6749#section-2.3.1).

CCAOnline only supports the HTTP Basic Authentication Scheme for Client Authentication.
CCAOnline does not support client credentials included in the request-body.

The followin example features a Basic Authorization Header that contains the client credentials (base64 encoded string for "someClientId:aSecret"). The resource owners password credentials are present in the body. 

```powershell 
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/token" -Method Post -Body @{ grant_type="ccaauth";username="alan";password="complete";authtype="SQLLogin"} -Headers @{"Authorization"="Basic c29tZUNsaWVudElkOmFTZWNyZXQ="}
```

## Resource Owner Password Grant

At the moment only OAuth2s ["Resource Owner Password Grant"](https://tools.ietf.org/html/rfc6749#section-4.3) is supported by the CoreDataService api.

**Attention**: Only CCA user credentials, that are configured with *SQL Authentication* can be used for the api.

## Token Endpoint

The Token Endpoint for requesting a bearer token is located under https://[ccaonlineRoot]/coredataservice/api/token .

## OAuth2 Bearer Token in Header

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen/42" -Headers @{"Authorization"="Bearer TOKEN";"Accept"="application/json"}
```
Besides the Bearer Token also the Accept Header with the value "application/json" has to be added to each request to the API.
# Pagination

Requests that return multiple items will be paginated to 15 items by default. You can request further pages with the `page` parameter. You can also request a larger page size (up to 100) by providing the `perPage` parameter.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?page=4&perPage=100"
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
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?orderBy=name-desc"
```
Returns a list of persons orderd by the `name` - property in descending order.

### Order by multiple properties

You can order by mulitple properties by providing the orderBy parameter multiple times.
The parameters will be resolved from left to right.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?orderBy=name-asc&orderBy=geburtstag-desc"
```

Returns a list of persons sorted by `name` (in ascending order) and `geburtstag` (in descending order).

Data Filtering 
==============

Filtering on the list api actions can be achieved by providing the property names as query parameter key and the filter expression as the associated value. The property names are case-insensitive.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?geburtstag=2016-05-19"
```

If an value is specified that does not match the property type, the api responds with `400 Bad Request`. 
Date values must be passed in the form `YYYY-MM-DD`.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?geburtstag=2016-05-19T10:25:00.928Z

HTTP/1.1 400 Bad Request
```

This query gets persons with geburtstags after the 1. Jan 1990

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?geburtstag=2016-05-19
```

This query gets persons who are customers

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?istkunde=true
```

Filtering by nested types is also possible. For example filtering Persons by the name of the "natuerlichePerson" the request has to look like the following:

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?natuerlichePerson.name=Testname
```

Filtering is although still only possible for properties which are part of the entity itself like name, birthday etc. For example filtering for sonstigePerson.sonstigPersonArt ist still not possible as here fields of the Table Anrede would have to be considered and not Person.

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
| boolean| eq                      |


However, the operation can be overridden with a second query parameter, with a key of the structure `<propertyName>+op`and the operation as value. The available alternative operations are also based on the property type. The filter operation values are case insensitive and will be normalized to lowercase.

|Datatype| Available Filter Operations| 
|--------|----------------------------|
| string | sw, cn, eq, gt, lt         |
| date   | eq, gt, lt                 |
| number | eq, gt, lt                 |
| boolean| eq                         |


This query gets persons with geburtstags after the 1. Jan 1990

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?geburtstag=1990-01-01&geburtstag+op=gt"
```

If an operation is specified that is not available in general, the api responds with `400 Bad Request`.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?geburtstag=1990-01-01&geburtstag+op=gaussian"

HTTP/1.1 400 Bad Request
```

If an operation is specified, but not the respective filter expression, the api responds with `400 Bad Request`.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?geburtstag+op=eq"

HTTP/1.1 400 Bad Request
```

If an operation is specified that is not available for the type, the api responds with (400) Bad Request.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?geburtstag=2016-05-19&geburtstag+op=sw"

HTTP/1.1 400 Bad Request
```

The identifier property cannot be part of a filter request, because gt and lt range queries are not supported and an exact-match-query can be accomplished with the respective details request. If it is specified the api responds with (400) Bad Request.
Example
```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?id=1000

HTTP/1.1 400 Bad Request
Content-Length: 11
X-Correlation-Id: 00000000-0000-0000-1c78-0080000000fc
```

### Multiple Filters

If multiple filters are passed they are always logically composed with AND.

This query gets people born on the 1. Jan 1990 named steve.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?geburtstag=1990-01-01&name=steve"
```

### Case Sensitivity

String filters work case-insensitive.

The `/personen` resource contains persons with names "Steve Buscemi" and "stephen colbert"

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?name=Ste"

HTTP/1.1 200 Ok

{
	...
	name: "Steve Buscemi",
	...
},
{
  ...
  name: "Stephen Colbert",
  ...
}
```

### Filter Ranges

To perform a range query with two specified bounds, the same property name is passed twice in the query parameters, with two different values. The parameters are position-sensitive, with the leftmost value being the lower bound and the rightmost value being the upper bound. The bounds are always interpreted as inclusive.

This query gets persons with a name beginning with A, everyone in between until the last one beginning with B.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?name=A&name=B"
``` 

For range queries, strings are matched with startsWith by default, and dates and numbers by their exact values. These operations cannot be overridden. If an operation is specified on a property that is part of a range query, the api responds with `400 Bad Request`.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?name=A&name=B&name-op=eq"

HTTP/1.1 400 Bad Request
```

If a property is specified more than twice, the api responds with (400) Bad Request.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?name=A&name=B&name=C"

HTTP/1.1 400 Bad Request
```

Two equal bound expressions are allowed, to ease bound computation on the client-side. In this case, the query will degenerate to a single-expression-query with the default range query operation applied.


This query gets persons with a full name starting with A.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?name=A&name=A"
```

This query gets persons with an ANP equal to a thousand

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?ANP=1000&ANP=1000"
```

Boolean values cannot be part of a range query.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen?isCompany=false&isCompany=true"

HTTP/1.1 400 Bad Request
```

# Multi-Language Support 


If the Accept-Language Header is set, the api will return dimensional data translated to the requested language, if a translation exists.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen/42" -Headers @{'Accept-Language'='en-us'}
```


# OMDS Version Support

## LandesCd

Currently the api supports the cocuntry codes for the OMDS Versions 2.9 and 2.10. The default are the ISO 3166-1 alpha-3 Codes from 2.10.
It is still possible to send/receive OMDS 2.9 LandesCodes. For this a Content-Type Attribute has to be added to the header with an additional "flavor" which indicates the required OMDS Version.

```powershell
Invoke-RestMethod -Uri "https://ccds.ccaedv.at/coredataservice/api/v1.210.2/personen" -Headers @{'Content-Type'='application/json+omds2.9'}
```

# Changelog
## 2.182 (1.210.16 - 14.11.2023)
* New endpoint:
  * PATCH /dokumente/{dokId} (Edits (via patch) a dokument per Id)
* Extended endpoints with new input parameter:
  * POST /sachRisiken/{sachRisikoId}/dokumente (extended with description)
  * POST /kfzRisiken/{kfzRisikoId}/dokumente (extended with description)

## 2.177 (1.210.15 - 25.07.2023)
* No modifications

## 2.176 (1.210.14 - 27.06.2023)
* New endpoint:
  * DELETE /personen/{perId}/beziehungen (Removes the relationships of the person)
* Extended endpoints with new input parameter:
  * POST /vertraege/{verId}/personRisiken (Links an existing Person to a contract as a risk / usable with array of person ids)
  * POST /vertraege/{verId}/kfzRisiken (Creates a KFZ to Vertrag {verId} / usable with array of KFZ Risiken)
  * POST /vertraege/{verId}/sachRisiken (Creates a NKP to Vertrag {verId} or only links them (existingRiskObject, existingRiskObjectCollection) / usable with array of NKP Risiken)

## 2.175 (17.05.2023)
* "vspKontoId" would not be set to NULL if "vermittlernummer" is not passed, but verDefaultKontoId will be used instead. Affected endpoints:
  * POST /personen/{perId}/antraege
  * POST /personen/{perId}/vertraege
  * POST /personen/{perId}/offerte

## 2.174 (18.04.2023)
* Added new endpoint: POST/personen/{perId}/Legitimationen
* Added new endpoint: PATCH/personen/{perId}/Legitimationen
* Added new endpoint: GET/personen/{perId}/Legitimationen
* Added new endpoint: POST/dokumente/{dokId}/unterschriftLinkErstellen/{perId}
* Removed "firmenbuchnummer" property from SonstigePersonModel

## 2.171 (30.11.2022)

* StatusCode is changed from number to string (OMDS Code of item) in PATCH/vertraege/{verId}/vertragsparten/{vspId}
* Only valid risks are transferred by the endpoints: GET/personen/{perId}/sachRisiken and GET/personen/{perId}/kfzRisiken. So the values of Gültig Ab (valid from) and Gültig Bis (valid to) are taken into account by the logic.
* Person's GET/PATCH/POST has changed/fixed. Rechtsform and UID handling is fixed in sonstige person. Firmenbuchnummer is removed from SonstigePerson and Socialversicherungsnummer is removed from natürliche person. Both can be found in person's identifizierung section just as before.

## 2.50

* Filtering for nested properties is now possible
* Filtering for properties build for the ViewModel like "istKunde" is now possible
* Changed default CountryCodes to OMDS Version 2.10
* Added CountryCode support for OMDS Version 2.10 and 2.9
* Bugfix regarding the creation of Adresses
* Delete PersonRelation operation added
* Delete Adresse from Person operation added
* Tools endpoint added with berechnejnp Operation

# Roadmap

## Releaseplan

| Test Release | Version | Release Date |
| ------------ | ------- | ------------ |
| 2017-07-13   | 2.46    | 2017-07-20   |
| 2017-08-03   | 2.47    | 2017-08-10   |
| 2017-08-29   | 2.48    | 2017-09-07   |
| 2017-10-18   | 2.50    | 2017-09-28   |
| 2019-11-15   | 2.100   | 2020-12-01   |
| 2020-02-17   | 2.102   | 2020-03-01   |


## Person Releated Operations

| OperationName | Enitites | description | CCAOnline Version |
|---------------| -------- | ----------- | ----------------- |
| createAdresse | Adressen | Creats a Adresse | **2.46** |
| getAdresse | Adressen | Gets an Adresse per Id | **2.46** |
| editAdresse | Adressen | Edits an Adresse per Id | **2.46** |
| getPersons | Person | Gets a List of Persons | **2.46** |
| createPerson | Person | Creats a Person | **2.46** |
| getPerson | Person | Gets an Person per Id | **2.46** |
| editPerson | Person | Edits an Person per Id | **2.46** |
| getPersonAddresses | Person | Gets addresses of a Person with Id {perId} | **2.46** |
| getPersonVertraege | Person | Gets Vertraege of a Person with Id {perId} | **2.46** |
| createPersonConnection | Person | Creats a connection between {perId} and {perId2} | **2.46** |

## Vertrag related Operations

| OperationName | Enitites | description | CCAOnline Version |
|---------------| -------- | ----------- | ----------------- |
| getVertraege | Vertrag | Gets a List of Vertraege | **2.46** |
| getVertrag | Vertrag | Get Vertrag By Id | **2.46** |
| createAntrag | Person | Creates an Antrag with Sparte | **2.44** |
| createVertrag | Person | Creates a Vertrag with Sparte | 2.47 |
| createOffert | Person | Creates an Offert with Sparte | 2.47 |
| editVertrag | Vertrag | Edits a Vertrag per Id | 2.47 |
| closeBook | Vertrag | Closed the Book of Vertrag {verId} | **2.44** |
| openBook | Vertrag | Open the Book of Vertrag {verId} | **2.44** |
| editVertragBuch | Vertrag | Set a Vertrag either to open or closed | 2.47 |
| setVertragXml | Vertrag | Sets the XML of Vertrag {verId} | **2.44** |
| createVSPToVertrag | Vertrag | Creats a VSP to Vertrag {verId} | 2.47 |
| getVSP | Vertrag | Gets a List of VSPs of Vertrag {verId} | 2.47 |
| getLZ | Vertrag | Get Leistungszeilen of Vertragssparte {spId} for Vertrag {verId} | 2.47 |
| editLZ | Vertrag | Edit Leistungszeile {lzId} of Vertragssparte {spId} for Vertrag {verId} | 2.47 |
| getVertragZusatzdaten | Vertrag, Zusatzdaten | Retrieves Zusatzdaten for the selected Vertrag {verId} | 2.47 |
| editZusatzdaten | Vertrag, Zusatzdaten |  Edits the additional datafields for Vertrag {verId}. The data is an update - setting values for additional datafields: **sets** the data if it was not set before **replaces** the data if it was already set\n  **removes** the current value if the value is `null`. All additional datafields that are not part of the patch request are ignored.| 2.47 |

## Risko related Operations

| OperationName | Enitites | description | CCAOnline Version |
|---------------| -------- | ----------- | ----------------- |
| getKFZRisiken | KFZRisiken | Gets a List of KFZRisiken | 2.48 |
| getKFZRisiko | KFZRisiken | Gets a KFZRisiko per Id | 2.48 |
| editKFZRisiko | KFZRisiken | Edits a KFZRisiko per Id | 2.48 |
| createKFZtoPerson | Person | Creats a KFZ to Person {perId} | 2.48 |
| getSachrisiken | Sachrisiken | Liste von Sachrisiken | 2.48 |
| getSachrisiko | Sachrisiken | Gets a Sachrisiko per Id | 2.48 |
| editSachrisiko | Sachrisiken | Edits a Sachrisiko per Id | 2.48 |
| createNKPtoPerson | Person | Creats a NKP to Person {perId} | 2.48 |

## Dokument related Operations

| OperationName | Enitites | description | CCAOnline Version |
|---------------| -------- | ----------- | ----------------- |
| getDokumente | Dokumente | Gets a List of Dokumente | 2.50 |
| getDokument | Dokumente | Gets a Dokument per Id | 2.50 |
| editDokument | Dokumente | Edits a Dokument per Id | 2.50 |
| downloadDokument | Dokumente | downloads a document per dokId | 2.50 |
| updateDokumentContent | Dokumente | Updates a documents content per dokId | 2.50 |
| createDokumentToPerson | Person, Dokumente | Creats a Dokument to Person {perId} | 2.50 |
| createDokumentToVertrag | Vertrag, Dokumente | Creats a Dokument to Vertrag {verId} | 2.50 |
