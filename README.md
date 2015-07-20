# REST API Design Guidelines

## Introduction

This document defines a set of design guidelines for an application programming interface (API). This document presents the conventions and design patterns to guide a REST API specification, along with rationale for the guidelines. As design guidelines, the content herein does not describe a specific implementation or particular API specification. Rather, the guidelines describe the properties to which an API specification and related implementation must adhere in order to be considered conformant to certain standard.

## Why is API design important?

* APIs can be among a company's greatest assets
* Customers invest heavily: buying, writing, learning
* Cost to stop using an API can be prohibitive
* Successful public APIs capture customers
* Can also be among company's greatest liabilities
* Bad APIs result in unending stream of support calls
* Public APIs are forever - one chance to get it right
* Thinking in terms of APIs improves code quality

## Characteristics of a good API

* Easy to learn. The API semantics must be intuitive
* Easy to use, even without documentation. There should not be different ways to achieve the same action
* Hard to misuse
* Easy to read and maintain code that uses it
* Sufficiently powerful to satisfy requirements
* Easy to extend
* Appropriate to audience

# The Process of API Design

## Gather requirements with a healthy degree of skepticism

* Often you'll get proposed solutions
  * Instead better solutions may exist
* Your job is to extract true requirements
  * Should take the form of use-cases
* Can be easier and more rewarding to build something more general

## Start with short spec - one page is ideal

* At this stage, agility trumps completeness
* Bounce spec off as many people as possible
  * Listen to their input and take it seriously
* If you keep the spec short, it’s easy to modify
* Flesh it out as you gain confidence
  * This necessarily involves coding

## Maintain Realistic Expectations

* Most API designs are over-constrained
  * You won't be able to please everyone
  * Aim to displease everyone equally
* Expect to make mistakes
  * A few years of real-world use will flush them out
  * Expect to evolve API

# General Principles

## API should be as small as possible but no smaller

* Functionality should be easy to explain
* API should satisfy its requirements
* When in doubt leave it out
  * You can always add, but you can never remove
* Conceptual weight more important than bulk
* Look for a good power-to-weight ratio

## Implementation should not impact API

* Implementation details
  * confuse users
  * inhibit freedom to change implementation
* Be aware of what is an implementation detail.
  * All tuning parameters are suspect
* Don't let implementation details “leak” into API

# Outline

API should meet following characteristics:

* Compactness (resource-based API is better than action-based API)
* Orthogonality (one end-point should do only one thing)
* Consistency (naming conventions of parameters, URI path structure, behavior of HTTP verbs, format and structure of reponses)
* Law of the least surprise (end-point should not have side-effects such as modification of another resource)

# Implementation recommendations

## Information and technical scope

REST interfaces are built around resources that define nouns. In the project management domain, these nouns include timesheets, task, human resources, etc., all of which have been rigorously defined as entities, attributes and relationships. REST API defines resources as compositions of entities, attributes and associations, called domain aggregates. The REST architectural style is a conventionbased approach to define application programming interfaces (APIs). HTTP , using the HTTP operations (GET, PUT, POST, DELETE, etc.), is used as the application protocol.

## HTTP Verbs

HTTP verbs communicate an action that should be taken against a resource. Depending on the verb, the request may include additional needed information in the body. Usually the following verbs are supported:

* **GET** - Returns existing resources.
* **POST** - Creates an individual resource. If successful, the URI to the new resource is returned in the “Location” HTTP header of the response.
* **PUT/PATCH** - Performs full or partial update of a resource. For a partial update, only the properties that are submitted will be updated on the target resource. The new representation of the entire resource is returned in the response body. According to the HTTP spec, you would use a verb named PATCH for partial update and PUT for full update. However not all web servers and moreover clients support PATCH so people have been supporting both partial updates with POST and PUT.
* **DELETE** - Deletes an existing individual resource.

## URIs

You should use nouns, not verbs (vs SOAP-RPC). For each resource, there are two base forms for the URI, one for a collection of resources and one for a specific resource in the collection. The collection is referred to by the pluralized name of the individual resource. A specific resource is referenced by the collection name, followed by a slash and the resource’s unique identifier. For example:

> */timesheets* refers to a collection of Timesheets

> */timesheets/923845* refers to a specific timesheet with an assigned identifier of "923845"

A collection of resources associated with another resource may be referenced in a single URI, using the pattern *“associatedResource/{id}/resource”*. For example:

> */timesheets/923845/activities* refers to the collection of activities associated with a specific timesheet

You may choose between *snake_case* (seems to be most popular), *spinal-case* or *camelCase* for attributes and parameters, but you should remain consistent. If you have to use more than one word in URL, you should use spinal-case (some servers ignore case).

## Query Operators

### Search

A REST API supports querying capabilities when searching a collection of resources. Query operators are applied to the query string using the following format: *{collectionURI}?{propertyName}{operator}{value}*.

> http://localhost:8080/api/timesheets?state=PENDING_FINAL

### Selectors

Selectors allow application developers to be more selective about how much data is returned in the resource representations. Fields are comma-separated.

| Parameter     | Description                                            |
|---------------|--------------------------------------------------------|
| includeFields | Limits the response to the fields listed.              |
| excludeFields | Limits the response to all fields except those listed. |

### Paging

When multiple records are being returned, the total count of all records is returned as an additional data member of the response body. The limit parameter can be used in the query string to set the maximum number of records returned. If no value is supplied, the limit parameter will default to all elements. The offset parameter can be set to specify how many records to skip when getting the result set. The default value for offset is 0. For example:

> http://localhost:8080/api/task?start=0&limit=3

## HTTP Responses

### Response Codes

A core tenet of RESTful APIs is embracing HTTP response codes to communicate status information. An API consumer should be able to inspect the HTTP response code and understand the status of its request. The following response codes are used when responding to requests.

| HTTP Response Code | Name                  | Reason(s)                                                                      |
|--------------------|-----------------------|--------------------------------------------------------------------------------|
| 200                | OK                    | Returned after a successful operation when a response contains a body.         |
| 401                | Unauthorized          | Returned if the access token is invalid. The response will not contain a body. |
| 404                | Not Found             | Returned if a resource is not found. The response will not contain a body.     |
| 500                | Internal Server Error | Returned if the server encountered an unexpected error during the operation.   |

### Errors

If an error occurs on the server, a 500 (Internal Server Error) code will be returned along with the addition of a message in the body, containing the error details. For example:

```json
{
   "error": {
       "type": "AuthError",
        "message": "Incorrect username or password [Incorrect username and/or password. Contact your help desk or your organization's internal Innotas Administrator for help with your userid or password.}]"
   }
}
```

## Bulk Load and Transfer

There are cases that require bulk updates of data. For example, user with role Manager may partially approve employee's ttimesheet. It will require client to update multiple activities of a timesheet. While this operation can be completed via a series of individual transactions, it would be less taxing on network traffic and overall execution time to execute a bulk request that would perform the entire batch. [Facebook Batch API](https://developers.facebook.com/docs/graph-api/making-multiple-requests) is one of the example of implementation of bulk operations. Client sends array of serialized requests, for example

```json
[
    {
        "method": "GET",
        "relativeUrl": "user/apiTokenForUsernamePasswordg?username=john.joe@example.com&password=blahblah"
    },
    {
        "method": "POST",
        "relativeUrl": "tasks",
        "body": "[\n        {\n            \"id\": 234533,\n            \"title2\": \"Motivational Speaking **\",\n            \"startDate2\": \"2001-06-05\",\n            \"targetDate2\": \"2001-06-08\",\n            \"completeDate\": \"2001-06-10\",\n            \"statusId2\": 232548\n        }\n]"
    }
]
```

Once both operations have been completed, server sends a response which encapsulates the result of all the operations. For each operation, the response includes a status code, header information, and the body. These are equivalent to the response you could expect from each operation if performed as raw requests against the Graph API. The body field contains a string encoded JSON object. For the above request, the expected response would be of the form:

```json
[
    {
        "code": 500,
        "body": "{ \"error\": { \"type\": \"AuthError\", \"message\": \"Incorrect username or password [Incorrect username and/or password. Contact your help desk or your organization's internal Innotas Administrator for help with your userid or password.}]\" } }"
    },
    {
        "code": 200,
        "body": "[\n  {\n    \"id\": 234533,\n    \"completeDate\": \"2001-06-10\"\n  },\n  {\n    \"id\": 234533,\n    \"completeDate\": \"2001-06-10\"\n  }\n]"
    }
]
```

## Authentication

Certain endpoints are available only to authenticated users. Client must obtain an access token to include in every call to the API. Token is passed either as query parameter or as http header.

## API versioning

You should make versioning mandatory in the URL at the highest scope (major versions). You may support at most two versions at the same time (native apps need a longer cycle). Default versioning shall be forbidden. Indeed, in case of changes in the API, the developers would not have the control over the impacts on calling applications.

> GET /v1/timesheets

# Used resources

* ["How to Design a Good API and Why it Matters"](http://lcsd05.cs.tamu.edu/slides/keynote.pdf) by Joshua Bloch (Principal Software Engineer at Google)
* [Ed-Fi® REST API Design Guidelines](http://www.ed-fi.org/assets/2013/11/Public-Ed-Fi-REST-API-Design-Guidelines-1.2.pdf)
* [Richardson Maturity Model](http://martinfowler.com/articles/richardsonMaturityModel.html)
* [How To Do RESTful Partial Updates](http://bitworking.org/news/296/How-To-Do-RESTful-Partial-Updates)
* [Best practice for partial updates in a RESTful service](http://stackoverflow.com/questions/2443324/best-practice-for-partial-updates-in-a-restful-service)
* [How to design a REST API](http://blog.octo.com/en/design-a-rest-api/)
