LivingSocial API Design Guide
=============================

## Version 1.0.0 - Published 2016 Mar 10

- [Quick Reference](QUICK.md)
- [FAQ](FAQ.md)

## Contents
- [Purpose](#purpose)
- [Overview](#overview)
    - [Why Have an API Design Guide?](#why-have-an-api-design-guide)
    - [Guiding Principles](#guiding-principles)
- [API Design Essentials](#api-design-essentials)
    - [Requests](#requests)
        - [URIs/Paths](#urispaths)
            - [Use Plural Resource Names](#use-plural-resource-names)
            - [Use Nouns, Not Verbs in the Base URI](#use-nouns-not-verbs-in-the-base-uri)
            - [Shorten Associations in the URI - Hide Dependencies in the Parameter List](#shorten-associations-in-the-uri---hide-dependencies-in-the-parameter-list)
        - [Uniform Interface with HTTP Verbs/Methods](#uniform-interface-with-http-verbsmethods)
    - [Resource Identifiers](#resource-identifiers)
        - [Do Not Use Database Table Row IDs as Resource IDs](#do-not-use-database-table-row-ids-as-resource-ids)
        - [Ruby SQUUID Implementations](#ruby-squuid-implementations)
        - [Clojure SQUUID Implementations](#clojure-squuid-implementations)
        - [Sample SQUUID Generator](#sample-squuid-generator)
    - [Versioning](#versioning)
        - [Why Versioning?](#why-versioning)
        - [Versioning Methods](#versioning-methods)
            - [Version in the HTTP `Accept` Header](#version-in-the-http-accept-header)
            - [Version in the URI](#version-in-the-uri)
        - [Versioning Schemes](#versioning-schemes)
            - [Breaking Changes](#breaking-changes)
            - [Non-Breaking Changes](#non-breaking-changes)
            - [Resources for Versioning Schemes](#resources-for-versioning-schemes)
        - [API Versioning Summary](#api-versioning-summary)
    - [Pagination](#pagination)
        - [Why Pagination?](#why-pagination)
        - [Pagination - Summary](#pagination---summary)
        - [Pagination Method](#pagination-method)
        - [Pagination - Leading the Consumer](#pagination---leading-the-consumer)
        - [Client Use of Pagination URIs](#client-use-of-pagination-uris)
    - [API-Only Sub-Domains](#api-only-sub-domains)
    - [Filter/Sort/Search](#filtersortsearch)
    - [Content Negotiation](#content-negotiation)
    - [Security](#security)
        - [Transport](#transport)
        - [Authentication/Authorization](#authenticationauthorization)
            - [Security (Authentication/Authorization)-Related Gems](#security-authenticationauthorization-related-gems)
        - [CORS (Cross-Origin Resource Sharing)](#cors-cross-origin-resource-sharing)
            - [CORS Implementations](#cors-implementations)
        - [Security References](#security-references)
    - [Responses](#responses)
        - [HTTP Status Codes](#http-status-codes)
        - [Error Handling / Messages](#error-handling--messages)
            - [Internal API Error Handling](#internal-api-error-handling)
            - [Consumer-Facing API Error Handling](#consumer-facing-api-error-handling)
            - [Response Validations](#response-validations)
            - [Error Handling References](#error-handling-references)
        - [Response Envelopes and Hypermedia](#response-envelopes-and-hypermedia)
            - [HAL Specification](#hal-specification)
            - [Developer Libraries for working with HAL](#developer-libraries-for-working-with-hal)
            - [Hypermedia Specifications Considered](#hypermedia-specifications-considered)
            - [Hypermedia References](#hypermedia-references)
        - [Response Meta Data](#response-meta-data)
    - [JSON](#json)
        - [Data Formatting](#data-formatting)
            - [Dates and Times](#dates-and-times)
                - [The Relationship Between [RFC 3339](http://tools.ietf.org/html/rfc3339) and [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601)](#the-relationship-between-rfc-3339-and-iso-8601)
            - [UTF-8](#utf-8)
            - [Currency and Fixed-Point Values](#currency-and-fixed-point-values)
        - [Third Parties and XML](#third-parties-and-xml)
    - [Consumer Tooling](#consumer-tooling)
        - [Ruby Consumer Tools](#ruby-consumer-tools)
    - [Artifacts](#artifacts)
        - [Documentation](#documentation)
        - [Executable Examples](#executable-examples)
    - [Performance](#performance)
        - [Consumer-Side](#consumer-side)
        - [Producer-Side](#producer-side)
            - [Condense JSON Response with HTTP Compression](#condense-json-response-with-http-compression)
            - [Conditional `GET` with Caching and `ETag`](#conditional-get-with-caching-and-etag)
        - [Performance References](#performance-references)
    - [Testing](#testing)
        - [Ruby Test Tooling](#ruby-test-tooling)
        - [Mobile Test Tooling](#mobile-test-tooling)
        - [Improve API Test Performance](#improve-api-test-performance)
    - [API Developer Guides - Implementing the API Design Guide on our Platforms](#api-developer-guides---implementing-the-api-design-guide-on-our-platforms)
        - [Why Do We Have Separate API Developer Guides?](#why-do-we-have-separate-api-developer-guides)
    - [Other API Design Guides](#other-api-design-guides)
        - [JSON References](#json-references)


## Purpose
The purpose of the LivingSocial API Design Guide is to provide
standards and best practices that all new LivingSocial APIs (both
internal and external) should follow. This guide is to help developers
and architects design and implement consistent and well-documented APIs
across our enterprise.

A basic knowledge of REST, HTTP, and JSON is assumed. We will provide
links to resources for those who want to learn more about the
fundamentals of these technologies.


## Overview
API Design has come of age, and has become a first-class citizen in the
enterprise. The principles and practices that we follow determine the
usability and overall quality of our APIs. We've seen web sites and
books on the topic, but why do we need an API Design Guide, and what
are the essentials of good API design?


### Why Have an API Design Guide?
An API Design Guide provides us with the following:

* APIs that have a consistent look-and-feel.
* APIs that act in a predictable/expected manner. Don't surprise the
  consumer.
* Take the guesswork out of designing and implementing an API by
  following the design practices and the accompanying developer guides.
* Streamlined tooling and documentation. Consistently designed APIs
  enable us to build tooling / automation around the defined standards
  (see Heroku's Prmd API document generation tool).

As we begin to externalize our APIs, they reflect on our company, and
become an extension of the LivingSocial brand and strategy.

This is a living document and subject to change and discussion. Please
file PRs against this repo to discuss changes and enhancements.

### Guiding Principles
Here are the guiding principles for designing an API and determining
the merit of each design practice:

* Use commonly accepted web standards (e.g., HTTP, JSON) when it makes sense to do so.
* Each API should be consistent with:
  * Other APIs in the enterprise.
  * Commonly accepted practices leveraged by other good APIs on the Internet (e.g., LinkedIn, Facebook, Twitter, etc.).
* It should be simple to consume and test.
* It must be pragmatic:
  * It must work and look the same across all of our core platforms - Ruby on Rails and Clojure.
     * To be cross-platform, avoid platform bleed-through whenever possible.
     * We will provide deeper implementation details in the Ruby on Rails and Clojure API Developer Guides.
  * It should perform well. None of these design practices should
    introduce a huge performance overhead. If we see this happening,
    then we need to re-visit that particular design practice. We
    recommend ["Microservices from Day One"](https://www.amazon.com/Microservices-Day-One-scalable-software-ebook/dp/B01MSXJ10K/ref=dp_kinw_strp_1)
    for specifics on how to monitor services.
  * Don't reinvent the wheel - leverage gems/libraries wherever possible.

## API Design Essentials
The remaining sections of this page cover the key areas to consider
when designing and implementing an API.


### Requests

#### URIs/Paths
The [URI (Uniform Resource
Indicator)](http://searchsoa.techtarget.com/definition/URI) / Path is
the path to a resource exposed by an API. Here are the key principles:

* URIs should be intuitive. This property, known as affordance:
  * Makes an API easy to use and understand.
  * Reduces the amount of documentation needed for an API.
* Keep URIs simple and short.
* Shorten associations/dependencies in the URIs.
* Use plural resource names.
* Use Nouns, not Verbs in the Base URI.
  * Use HTTP Verbs - GET, POST, PUT, DELETE, and PATCH - to indicate the operation on a resource.

##### Use Plural Resource Names
Resource names should be plural, for example if we're exposing
Inventory Items, then the Base URI should look like:

```
/inventory_items
```

unless the resource is a singleton, for example, the overall status of
the system might be `/status`.

##### Use Nouns, Not Verbs in the Base URI
Never put a Verb in the Base URI. Rather than something like `/get_inventory_by_id`, we use the following URI with an HTTP GET:

```
/inventory_items/4
```

We'll cover appropriate HTTP Verb usage in the HTTP Verbs section below.

##### Shorten Associations in the URI - Hide Dependencies in the Parameter List
Resources are related to one another, and eventually they are stored in a database with associations between the tables. In the early days of REST, there were URIs that looked like:

```
customers/1/orders/54/inventory_items/5900
```

This is bad practice because:

* It reveals too much detail about the relationship between customer, order, and inventory items.
* The associations in the underlying database bleed through to the API.
* The API becomes brittle and difficult to change because of tight coupling between the URI and the database.

Instead, shorten the association in the URI and hide the nested data dependencies in the parameter list. Most URIs should go no deeper than the following:

```
/resource/{identifier}/resource
```

With this structure in mind, the new path now looks like:

```
customers/1/orders?order_id=54&inventory_item=5900
```

#### Uniform Interface with HTTP Verbs/Methods
The following table shows the standard HTTP Verbs/Methods to act on resources. Use this approach rather than cluttering the URI with verbs.

| HTTP Verb / Method | Action             |
|:-------------------|:-------------------|
| `GET`              | Read               |
| `POST`             | Create             |
| `PUT`              | Update (full)      |
| `PATCH`            | Update (partial).  |
| `DELETE`           | Delete             |


### Resource Identifiers
Resources should use Sequential UUIDs (SQUUIDs).

We chose SQUUIDs because they limit fragmentation of indexes and eliminate [performance problems in MySQL](http://www.percona.com/blog/2007/03/13/to-uuid-or-not-to-uuid/). __SQUUIDs are stored as binary(16)__

_SQUUIDs should be represented in string form as lowercase.  This is so that they can easily be joined across tables in the data warehouse._

#### Do Not Use Database Table Row IDs as Resource IDs
Auto-incrementing integer database row identifiers should not be used as Resource IDs or exposed in any way through the API.

Exposing DB table "auto-increment" row IDs as Resource IDs leaks implementation details outside the API. Row IDs couple the resource too closely to the underlying persistence engine and schema, and can cause headaches when the owning service might need to migrate away from a particular persistence engine or structure, or when clustering of several DBs becomes necessary. IDs not generated by autoincrementing column IDs allow for more flexibility.

#### Sample SQUUID Generator
```
def squuid
  This is basically the implementation of SecureRandom.uuid ...
  ary = SecureRandom.random_bytes(16).unpack("NnnnnN")
  ... but we replace the high-order 32 bits with the current time.
  ary[0] = Time.now.to_i
  ary[2] = (ary[2] & 0x0fff) | 0x4000
  ary[3] = (ary[3] & 0x3fff) | 0x8000
  "%08x-%04x-%04x-%04x-%04x%08x" % ary
end
```

### Versioning

#### Why Versioning?
API Versioning is an important aspect of API design because it informs the consumer about an API's capabilities and data. Consumers use the version number for compatibility.

#### Versioning Methods
Here are 2 of supported methods of API versioning:

* **Version in the HTTP Accept Header.** (*this is our preference*)
* Version in the URI.

We currently have some APIs that use Header-based versioning, and some that use URI-based versioning, in addition to deployed mobile apps that invoke URI-versioned APIs. Here is the direction we would like to take:

* Put the API Version in the HTTP Header for all new development.
* To avoid negatively impacting the current APIs under development, they can continue to use URI-based versioning for the time being.
* As opportunities arise, refactor all URI-versioned APIs to Header-based APIs.

##### Version in the HTTP `Accept` Header

```
GET
/inventory_items/4
Accept: application/vnd.livingsocial.v1+json
```
Here are the pros when putting the version in the HTTP `Accept` Header:

* The version is a representation of a resource, and (according to the [HTTP specification](https://tools.ietf.org/html/rfc2616)) this information goes in the HTTP Accept Header.
* It leverages a mechanism already provided by the HTTP specification. It's good to follow standards, and there's no need to invent something else.
* It is intellectually consistent with [Roy Fielding's dissertation on REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm).
* It supports content-based load balancing provided by web servers such as nginx.

Here are the cons:

* It's harder to test because you can't test it from a browser - you can't see the HTTP Headers without additional tooling.
  * But RESTful client plugins take care of this issue, and they are widely available for all major browsers. Plus, there are several good standalone RESTful API test tools.

##### Version in the URI
Many APIs put the version in the URI. Here's an example:

```
GET
/v1/inventory_items/4
```

Here are the pros when putting the version in the URI:

* It works well and is widely used by many of the major APIs on the Internet (e.g.,  Twitter, Yammer, Facebook, Google, etc.).
* It's simple and easy to read.
* It supports testing from a web browser.

Here are the cons:

* A URI identifies the location of resource, and the URI shouldn't change just because the data changes. A URI shouldn't change over time. A resource is a resource.
* A new URI should only be introduced when a new resource is created, not when it's representation or version changes.

#### Versioning Schemes
A pragmatic versioning scheme completes the overall API versioning strategy and addresses how to format the version number, when to upgrade and when to retire a version:

* We will base our approach on Semantic Versioning ([semver](http://semver.org/)) and Fear-Driven Versioning ([ferver](https://github.com/jonathanong/ferver)).
* We will only use a Major and Minor version in the form `x.y`, where `x` >= 1 and represents the Major version number, and `y` >= 0 and represents the Minor version number.
  * We never have a Major version of 0 because it makes the API appear unstable.
* There is no need to modify either the Major or Minor version for Non-Breaking changes.
* We change the Major version when:
  * There are multiple Breaking Changes.
  * There has been a significant change in the overall direction or philosophy of the API.
* We change the Minor version when there are 1 or more Breaking Changes.
* We retire a version after a certain period of time to give consumers/clients a chance to upgrade. For example, when we upgrade the Inventory Items API to version 2, we allow both version 1 & 2 to run concurrently for a certain period of before we sunset version 1.

##### Breaking Changes
A Breaking Change is any change to an API that could break a contract and cause consumer/client invocations of the API to fail. When changing an API, there are several things to think about when determining if a new version is needed or if the current version is still going to work for the client:

* Deprecating a feature (e.g., resource, Path, HTTP Verb, or JSON property).
* Refactoring a non-trivial portion of the API implementation.
* When a field's data type changes (e.g., the price changes format from string to float or vice-versa).
* When the format for a field changes (e.g., date formatting, currency).
* Updating an external dependency that the API relies on.
* Changing the Security model.

##### Non-Breaking Changes
A Non-Breaking Change is a change to the API that does not break a contract nor does impact the consumer/client. Here are some examples:

* Adding a new feature (e.g., Resource, Path, HTTP Verb, or JSON property).
* Upgrading documentation.

##### Resources for Versioning Schemes
Here are a few resources for dealing with the above issues:

* Semantic Versioning ([semver](http://semver.org/)). Many APIs and libraries use semver, which uses the following format for a version: `MAJOR.MINOR.PATCH`
* [Semver has failed us](http://www.jongleberry.com/semver-has-failed-us.html). A nice article on the shortcomings of semver, and shows how to improve it.
* Fear-Driven Versioning ([ferver](https://github.com/jonathanong/ferver)). This is based on semver, and makes some modifications. ferver has the notion of "breaking changes" and provides some guidelines there. They favor this version format: `MAJOR.MINOR`
* [Heroku's API Compatibility Policy](https://devcenter.heroku.com/articles/api-compatibility-policy) provides high-level guidelines on when to change and retire versions.

#### API Versioning Summary
Based on the research and discussions above, here are the high-level recommendations for versioning an API:

* Every API must have a version.
* Every API invocation must specify a version number.
* Versioning Methods:
  * All new APIs will put the version in the HTTP Accept Header.
  * All APIs currently under development can continue to use URI-based versioning for now, but will be refactored to use HTTP Accept Header in the near future.
* Versioning Schemes:
  * We will use a Major and Minor version in the form `x.y`, where `x` represents the Major version number, and `y` represents the Minor version number.
  * We change Major and Minor versions based on the degree and severity of the change to the API contract.

### Pagination

#### Why Pagination?
An API must be able to control/gate the amount data returned in the response so that the Consumer is able to handle the volume of data. If an API returns all instances of a given resource (e.g., inventory items, etc.) it could easily overrun the memory and processing capacity of the consumer. Pagination helps control the volume of data returned and makes it easier for the Consumer to process.

#### Pagination - Summary
Here are the high-level recommendations for API Pagination:
* Use the Facebook-style of Pagination that uses an `offset` and `limit` as follows: `/inventory_items?offset=543&limit=25`
  * The `offset` has semantic meaning. It is a sortable field/column based on the most optimal way to access the resource's data.
  * The `limit` has no semantic meaning. It is a simple integers that have no relation to the resource's data.
* Lead the Consumer through the API with links/URIs in the JSON response that show how to navigate.
  * Consider [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-06) as a guide to specify these links/URIs.

#### Pagination Method
For our APIs, the `offset` is semantic - it could be a UUID, ID, a Date, etc. that is sortable. This semantic offset approach gives the API Producer (i.e., the developer who creates the API) a chance to choose an optimal way to do an offset into a result set based on a sortable field/column based on the most efficient way to access the resource's data. But the consumer will not provide the `offset` and `limit`; instead, the API itself provides this information to the Consumer.

#### Pagination - Leading the Consumer
An API should lead the consumer through the API invocations to help them reach a goal. In this case, we want to lead the consumer forward and backward through the result set rather than making them guess or keep track of where to go next. Each API should provide hyperlinks that provide URIs to the next and previous pages. This is an example of [HATEOAS (Hypertext as the Engine of Application State)](http://restcookbook.com/Basics/hateoas/), and here's what these hyperlinks might look like in a JSON response:

```
GET
/inventory_items?offset=100&limit=25
{
   "_links": {
     "self": {
       "href": "/inventory_items?offset=543&limit=25"
     },
     "next": {
       "href": "/inventory_items?offset=768&limit=25"
     },
     "prev": {
       "href": "/inventory_items?offset=123&limit=25"
     }
   },
   "items": [
     {
       "name": "Washington Nationals Game",
       "type": "Sporting Event",
       "price": "$35.00"
     }
     ...
  ]
}
```

In the above example, the `"_links"` property provides links to the consumer to help them navigate through the result set:

* `"self"` - The URI that was invoked.
* `"next"` - The next group of results.
* `"prev"` - The previous group of results.

This is a very simple implementation of [HAL (JSON Hypermedia API Language)](hhttps://tools.ietf.org/html/draft-kelly-json-hal-06). For further information, please visit the [HAL site](http://stateless.co/hal_specification.html).

#### Client Use of Pagination URIs
For client apps to use paginated APIs, it is necessary to embed the appropriate data from `_links` into the client views. For example, when the client displays a view containing paginated data, the `next` link must be embedded into the view so that the client web app knows where to fetch the next page of API data. Clients should avoid trying to construct new URLs to the API, but rather depend on the `next` and `previous` links returned by the API.

It is also recommended to encrypt the pagination URIs embedded in client views to prevent bad actors from making arbitrary calls to our backend services.

Given the example results in [Leading the Consumer](#pagination---leading-the-consumer), the client view might generate a links in its view like this ERB template example:
```ruby
<a href="/results?page=<%= encryptor.encrypt_and_sign("/inventory_items?offset=543&limit=25") %>">Next Page</a>
```

The controller that renders the `/results` view can use the value of the `page` parameter to fetch the next page of data from the API.


### API-Only Sub-Domains

If your API will be publicly accessible, contact Architecture to have a
discussion about hosting it under `api.livingsocial.com`, as there are
many potential benefits to doing so. 

### Filter/Sort/Search
Here are some general guidelines:
* Keep it simple.
  * Just use simple HTTP parameters.
  * Use a separate field for each filter/sort/search parameter.
* Avoid using a delimited set of fields (this anti-pattern is borrowed from the [Pearson API Guide](https://github.com/tfredrich/RestApiTutorial.com/raw/master/media/RESTful%20Best%20Practices-v1_2.pdf)):

  ```
  GET http://www.example.com/users?filter="name::bill|city::denver|title::developer"
  ```

  * The above is overly complex because it requires a `::` delimiter between the `filter` fields.

* Filter/sort/search on fields that are sortable field/column based on the most optimal way to access the resource's data (mot likely on a sortable/indexable column in the underlying database). Please see [Pagination](#pagination) for further details.
* For filter and search, there's no need for a `filter` or `query` because this is implied by the HTTP parameters.
* For sort, we should use an explicit `sort` parameter (to instruct the API to sort the response) as follows:
```
https://catalog-service.ls.net:443/offers.json?sort=city_id
```


### Content Negotiation
[Content Negotiation](http://en.wikipedia.org/wiki/Content_negotiation), part of the HTTP specification, is a mechanism that enables an API to serve a document/response in different formats (e.g., XML, JSON, etc.). Based on current industry best practices, we prefer JSON.

An API consumer can specify that they expect JSON in the response by using the HTTP Accept Header or by adding an extension to the URI. Here's how to specify the JSON [MIME type](http://en.wikipedia.org/wiki/Internet_media_type) in the HTTP `Accept` Header:

```
Accept: application/json
```

If JSON wasn't specified in the HTTP `Accept` Header, here's how to do this with a `.json` URI extension (Rails provides this by default).

```
.../cities/nearby/zipcode/1234.json
```

The HTTP `Content-Type` Header is the other header involved in Content Negotiation, and works as follows:
* An API consumer specifies that it provides JSON content in the request body (for `POST` and `PUT`).
* An API specifies that it provides JSON content in the response.

Here's how to specify the JSON [MIME type](http://en.wikipedia.org/wiki/Internet_media_type) in the HTTP `Content-Type` Header:

```
Content-Type: application/json
```

* API Producers:
  * For Rails-based APIs, Rails sets JSON MIME type headers by default.
  * For Clojure-based APIs, Clojure Ring Middleware sets JSON MIME type headers on behalf of the API (so that the handler doesn't have to deal with this).


### Security
Each API must address the following security concerns:
* Transport
* Authentication/Authorization
* CORS (Cross-Origin Resource Sharing)

#### Transport
Always use HTTPS to communicate with and between APIs. A best practice is to use TLS by default.

For internal APIs hosted in our IAD data center, this usually adds a performance penalty and can be dropped. As we move to AWS hosted services in 2016 and beyond, be sure to watch for this.

The goal here is confidentiality by hiding/encrypting sensitive information so that unauthorized 3rd parties cannot see the textual data transmitted between an API Producer and Consumer.

#### Authentication/Authorization
The purpose of Authentication is to validate a service consumer (i.e., subject/identity), which could be a:
* User
* A Mobile or Web application
* Another Service (either internal or external)

Authorization ensures that a subject/identity has permission to use/access services and resources in a system. Authorization usually occurs as part of or after Authentication. In this case, Authorization ensures that a service consumer has the right to access and use an API.

#### CORS (Cross-Origin Resource Sharing)
Web and mobile applications can only make HTTP requests to the site (i.e., domain) they're currently displaying. For example, if the UI is running on `www.myapp.com`, it can't perform an HTTP request against `www.yourapi.com`. There are 2 ways to handle this:
* [JSONP](http://en.wikipedia.org/wiki/JSONP) - JSONP (aka "JSON with Padding") provides a way to make an HTTP request against a different domain:
  * The API returns JavaScript rather than JSON.
  * The API consumer uses JavaScript to parse and interpret the response.
  * This is possible because browsers don't enforce the same-origin policy within `<script>` tags.
* [Cross-Origin Resource Sharing (CORS)](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) - CORS
  is recommended over JSONP because CORS is a [W3C standard](http://www.w3.org/TR/cors/) and supported by all modern browsers.

CORS defines a mechanism in which an API and its Consumers can collaborate to determine if it's safe to  allow the cross-origin request by leveraging HTTP Headers.

The `Origin` is the only relevant CORS-related HTTP Request Headers, and it indicates the origin (i.e., domain) of the cross-site access request. Upon receiving an HTTP Request, an API can check the `Origin` header's value (i.e., the originating domain - `myoriginatingdomain.com`, for example) against a whitelist of allowed domains. If the originating domain is __not__ found in the whitelist, then the API should return a 403 (`Forbidden`) Status Code.

Otherwise, if the originating domain is found in the whitelist, then the API performs the request and populates the following CORS-related HTTP Response Headers:

| HTTP Response Header | Description | Example |
| -------------------- | ----------- | ------- |
| `Access-Control-Allow-Origin` | Indicates whether a resource can be shared based by returning the value of: the `Origin` request header, `*`, or `null`. Even though `*` is allowed by the CORS spec, don't use it because this implies that every domain is acceptable. Use the value of the `Origin` request header instead. | `myoriginatingdomain.com` |
| `Access-Control-Allow-Credentials` | Indicates whether the response to a request can be exposed when the omit credentials flag is unset. When part of the response to a preflight request it indicates that the actual request can include user credentials.  | `true` |
| `Access-Control-Allow-Headers` | Indicates which HTTP headers are safe to expose to the API of a CORS API specification. | `Authorization` |
| `Access-Control-Allow-Methods` | Indicates, as part of the response to a pre-flight request, which HTTP Methods can be used during the actual request. | `GET`,  `POST`, `PUT`, `DELETE` |

##### CORS Implementations
There are a couple of good implementations:
* [rack-cors](https://github.com/cyu/rack-cors)

To have common/reusable CORS functionality in a single place across our APIs (i.e., the DRY Principle), we should consider:
* Populating the `Origin` Request Header from Mobile app and web applications.

#### Security References
* See "Securing Services", "Authentication", "Authorization", "Transport Security", and "Application Security" sections in the [Pearson eCollege RESTful Services Best Practices Guide](https://github.com/tfredrich/RestApiTutorial.com/raw/master/media/RESTful%20Best%20Practices-v1_2.pdf). Also see the "Handling Cross-Domain Issues" section.
* [CORS Specification](http://www.w3.org/TR/cors/)
* [Enable CORS](http://enable-cors.org/)
* See "Always use HTTPS" and "CORS" sections in [18F API Standards](https://github.com/18F/api-standards).


### Responses
When a Consumer invokes an API, one of three things can happen:
* Success - everything worked properly.
* Consumer Error - The Consumer (i.e., client) used bad data or there was a malformed request.
* API Error - The API (i.e., Producer, server) had a processing error.

Each API needs to handle errors and convey an appropriate status and error message in the HTTP Response.

#### HTTP Status Codes
Here is a list of the most common [HTTP Status Codes](http://www.restapitutorial.com/httpstatuscodes.html), their usage, and contextual meaning.

| HTTP Status Code | Applicable HTTP Verb / Method | Meaning                                                                                                                                                                                                   |
|:-----------------|:------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 200              | GET / PUT / DELETE            | `OK`. The API processed the request properly without error.                                                                                                                                               |
| 201              | POST                          | `Created`. API successfully created the Resource. The URI of the newly created Resource should go in the Location Header of the Response.                                                                 |
| 202              | POST / PUT / DELETE           | `Accepted`. The Request has been been received, has not been completed, and will be processed later.                                                                                                      |
| 204              | POST / PUT / DELETE           | `No Content`. API successfully processed the request, and is not returning data.                                                                                                                          |
| 301              | GET / PUT / DELETE            | `Moved Permanently`. This and all future requests should be directed to the new URI given in the Response.                                                                                                |
| 304              | GET                           | `Not Modified`. The resource hasn't changed since the previous GET request. Consumer must provide the following Headers: `Date`, `Content-location`, `ETag`.                                              |
| 400              | ALL                           | Bad Request. Due to Malformed URI or invalid data in the request parameters or body.                                                                                                                      |
| 401              | ALL                           | `Unauthorized`. Indicates that the Consumer has not provided appropriate credentials or that the supplied credentials are invalid.                                                                        |
| 403              | ALL                           | `Forbidden`. Similar to 401 (Unauthorized), but the API chose not to respond. This could be due to the fact that the HTTP Verb for the Request URI is not allowed.                                        |
| 404              | GET / PUT / DELETE            | `Not Found`. The API couldn't find the resource specified by the Request URI.                                                                                                                             |
| 409              | PUT                           | `Conflict`. The Request can't be processed due to an edit conflict on a resource. This could be caused by a race condition on the data to multiple updates from different consumers on the same resource. |
| 410              | ALL                           | `Gone`. The resource is unavailable and will not be available again.                                                                                                                                      |
| 500              | ALL                           | `Internal Server Error`. The request couldn't be processed due to a server-side processing exception.                                                                                                     |
| 501              | ALL                           | `Not Implemented`. The request is valid, but functionality has not been implemented for the HTTP Verb and Request URI.                                                                                    |
| 503              | ALL                           | `Service Unavailable`. The API (i.e., server) is temporarily unavailable. The server could be overloaded or down for maintenance.                                                                         |


#### Error Handling / Messages
Error Handling / Messaging requirements are different for Internal APIs and Consumer-Facing APIs.

##### Internal API Error Handling
[HTTP Status Codes](#http-status-codes) are sufficient for expressing statuses and error conditions for APIs that are consumed by other APIs. Internal APIs could handle errors (especially uncaught exceptions) and return the information in the JSON response as follows:

```
{
  "message": "Error message high-level description.",
  "exception": "[detailed stacktrace or error message]"
}
```

In production, we shouldn't have to put error messages in the response.

##### Consumer-Facing API Error Handling
Consumer-facing applications (both web and mobile) -  that consume an API and display error information to the end user - need more contextualized information is needed for consumer-facing applications. In this case, errors that include highly contextualized (localized, relevant to the action they're taking, etc) error messages would provide a much better experience to the user when there is an error.

Based on the [JSON API Error Formatting Guidelines](http://jsonapi.org/format/#errors), APIs used by consumer-facing applications __could__ provide the following fields in the error response:
* Required:
  * `code` - An application-specific error code, expressed as a string value.
* Optional:
  * `title` - A short, human-readable summary of the problem. It __should not__ change from occurrence to occurrence of the problem. This would be non-localized (i.e., English) because developers are the audience this.
  * `details` - A human-readable explanation specific to this occurrence of the problem. This would also be non-localized (i.e., English).
  * `user_message` - Localized error message, to be conveyed to the (mobile) end user.

Here's an example error message:

```
{
  "errors": [
    {
      "code": "unrecoverable_error",
      "title": "The flux capacitor disintegrated",
      "details": "Hold on, the end is nigh.",
      "user_message": {
        "default": "OMG, panic!",
        "en-GB": "Keep a stiff upper lip",
        "de-DE": "Schnell, schnell!!"
      }
    }
  ]
}
```

##### Response Validations

The `ls-api_validation` gem automates response validations. The gem looks at either your Swagger documentation or published [JSON Schema](https://github.com/ruby-json-schema/json-schema) documents to ensure your responses live up to their promises. In development `ls-api_validation` will raise an exception if the schema is invalid and in production it will validate a percentage of responses (1% by default) and log statuses. These actions are also configurable so you can set these to match your use cases.

##### Error Handling References
Here are some error handling examples from other API Design Guides:
* ["Generate structured errors" from the Heroku API Guide](https://github.com/interagent/http-api-design#generate-structured-errors).
* ["Error handling" in the 18F API Standards](https://github.com/18F/api-standards#error-handling).
* [JSON API Error Formatting Guidelines](http://jsonapi.org/format/#errors).

#### Response Envelopes and Hypermedia
Core RESTful principles are great at describing which HTTP verb to use and how to design manageable URIs. But there are two problems that need to addressed in the Response:
* Links between API endpoints/resources.
* Standardized response structures.

##### HAL Specification
Of the several Hypermedia specifications we reviewed (see [Hypermedia Specifications Considered](#hypermedia-specifications-considered)), we prefer [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-06) because:
* It's simple and lightweight.
* The links (`_links`) and data (`_embedded`) don't impose heavy structural or formatting changes to the response data. The JSON response data still refelects the intent/structure of the original resource.
* Much of the specification is optional, so it's easy to comply with its requirements.

Here's an example:

```
   GET /orders/523 HTTP/1.1
   Host: example.org
   Accept: application/hal+json

   HTTP/1.1 200 OK
   Content-Type: application/hal+json

   {
     "_links": {
       "self": { "href": "/orders" },
       "next": { "href": "/orders?page=2" },
       "find": { "href": "/orders{?id}", "templated": true }
     },
     "_embedded": {
       "orders": [
         {
           "_links": {
             "self": { "href": "/orders/123" },
             "basket": { "href": "/baskets/98712" },
             "customer": { "href": "/customers/7809" }
           },
           "total": 30,
           "currency": "USD",
           "status": "shipped"
         },
         {
           "_links": {
             "self": { "href": "/orders/124" },
             "basket": { "href": "/baskets/97213" },
             "customer": { "href": "/customers/12369" }
           },
           "total": 20,
           "currency": "USD",
           "status": "processing"
         }
       ]
     },
     "currentlyProcessing": 14,
     "shippedToday": 20
   }
```

In the above document:
* The `_links` section at the root contains links for pagination (`self`, `next`) and search (`find`).
* The `_embedded` resources section contains:
  * An `orders` array, whose elements each contain:
    * Order properties: `total`, `currency` and `status`
    * Links to associated resources: `basket` and `customer`

Please note that the `_links` object is not required by each `order` object - it is optional in that context.

Here are the requirements for a simple valid HAL document:
* The Response Header must include the following MIME type: `application/hal+json`
* The root object __must__ be a Resource Object, which is one of:
  * `_links` - Optional. Contains links to other resources. The `links` object has several optional properties. Here's the main property, which is required:
    * `href` - Its value is either a [URI](https://tools.ietf.org/html/rfc3986) or a [URI Template](https://tools.ietf.org/html/rfc6570).
  * `_embedded` - Optional. Contains embedded resources, which could be a full or partial version of
     the representation served from the target URI.

HAL is simple and works well, and we should use it for every JSON response from our APIs. Here are a couple of example use cases for HAL:
* [Pagination](#pagination---leading-the-consumer---proposed)
* Reduce coupling and payload size for deep object hierarchies. For example, an `order` could have links to each `lineItem` that it contains rather than embedding the full content for the `lineItem` objects within the `order`.

##### Developer Libraries for working with HAL
Here's a list of [developer libraries for working with HAL](https://github.com/mikekelly/hal_specification/wiki/Libraries). The most promising libraries are:
* Rails:
  * [ROAR (Resource-Oriented Architectures in Ruby)](https://github.com/apotonick/roar/blob/master/lib/roar/json/hal.rb)
  * [Hyperresource](https://github.com/gamache/hyperresource)
  * [Hyperclient](http://codegram.github.io/hyperclient/)
  * [Hal-Client](https://github.com/pezra/hal-client)
* Clojure:
  * [halresource](https://github.com/cndreisbach/halresource)
* JavaScript
  * [hyperagent.js](http://weluse.github.io/hyperagent/)

The above list is a small subset of the [available HAL libraries](https://github.com/mikekelly/hal_specification/wiki/Libraries), so individual development teams should research and try out several options to find what works best for them.

##### Hypermedia Specifications Considered
There are several Hypermedia specifications that enable developers to link API responses and standardize API response formats:
* [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-06)
* [JSON API](http://jsonapi.org/)
* [JSON-LD](http://json-ld.org/)
* [Siren](https://github.com/kevinswiber/siren)
* [Collections+JSON](http://amundsen.com/media-types/collection/examples/)

We reviewed the above options for a consistent envelope that standardizes:
* Data - The payload (the main data in the Response).
* Links - What can the consumer do next?
* Metadata - Documentation without requiring the consumer to read through the API docs.

In general, we found that all of the above met these needs, but there were issues with the Data and Metadata structures required by most of these specifications:
* Data - If this imposes too many structural and formatting changes to existing JSON Responses, then it's too heavy and obfuscates the meaning/structure of the resource data.
* Metadata - The embedded documentation required by these specifications seems too heavy to include in a JSON Response. The developer consuming the API can just read the API documentation.

##### Hypermedia References
* [On choosing a hypermedia type for your API - HAL, JSON-LD, Collection+JSON, Siren, Oh My!](http://sookocheff.com/posts/2014-03-11-on-choosing-a-hypermedia-format/)
* [Please, no more generic hypermedia types](http://www.bizcoder.com/please-no-more-generic-hypermedia-types)
* [The Hypermedia Debate](http://www.foxycart.com/blog/the-hypermedia-debate#.VXYJnVxViko)
* [JSON-LD and Why I Hate the Semantic Web](http://manu.sporny.org/2014/json-ld-origins-2/)
* [HAL specification](https://tools.ietf.org/html/draft-kelly-json-hal-06)
* [HAL site](https://github.com/mikekelly/hal_specification/wiki/Libraries)
* [HAL GH Repository](https://github.com/mikekelly/hal_specification)
* [HAL Primer](https://apigility.org/documentation/api-primer/representation-formats)
* MuleSoft:
  * [API Best Practices: Hypermedia (Part 1)](http://blogs.mulesoft.com/api-best-practices-hypermedia-part-1/)
  * [API Best Practices: Hypermedia (Part 2)](http://blogs.mulesoft.com/api-best-practices-hypermedia-part-2/)
  * [API Best Practices: Hypermedia (Part 3)](http://blogs.mulesoft.com/api-best-practices-hypermedia-part-3/)
* [Hypermedia API on Rails](http://guillecarlos.com/hypermedia-api-on-rails-04-24-2013.html)

#### Response Meta Data
Occasionally there will be a need to return meta-data about a response that doesn't belong in the JSON for the individual objects in the response. This data should go in a hash in a top level `meta` attribute. For instance, catalog-service can return offers using one of many different 'representations'. To include the name of the `representation` used for this particular response we might return something like this:

```
GET /offers/{guid}.json
{
  "meta": {
    "representation": "tile",
  },
  "offer": {
    "id": 123,
    ... other attrs ...
  }
}
```

```
GET /offers.json?some=search_query
{
  "meta": {
    "representation": "tile",
  },
  "offers": [ ..list of offers... ],
  ...
}
```

### JSON
Our APIs and Kafka messages use JSON because it is the format of choice for most modern Web APIs.
JSON provides interoperability and cleanly converts to programming constructs (i.e., objects, arrays, key/value pairs) in most development languages.

#### Data Formatting

##### Dates and Times
We use [RFC 3339](http://tools.ietf.org/html/rfc3339) to represent Dates and Times in [UTC (Coordinated Universal Time)](http://en.wikipedia.org/wiki/Coordinated_Universal_Time) format for interoperability. Here's an example of an RFC 3339-compliant date/time:

```
2008-09-08T22:47:31Z-05:00
```

Here's an example of the date in a JSON document:

```
{
  ...
  "created_at": "2008-09-08T22:47:31Z-05:00"
  ...
}
```

Here are the formatting guidelines:
* `T` separates the date from the time.
* `Z` separates the time from the [UTC offset](http://en.wikipedia.org/wiki/UTC_offset), which indicates the Time Zone.
* Dates have the following format - `YYYY-MM-DD`, where:
  * `YYYY` is a 4-digit year.
  * `MM` is a 2-digit month. Prefix with a `0` if the month # is < 10.
  * `DD` is a 2-digit day of the month.
* Timestamps have the following format - `hh:mm:sss:sss`, where:
  * `hh` - 2-digit hour based on a 24-hour clock.
  * `mm` - 2-digit minutes.
  * `ss` - 2-digit seconds.
  * `sss` - 3-digit fractional seconds.
  * UTC Offsets have the following format - `?[hh]:[mm]`, where:
    * `?` indicates whether the offset is ahead (`+`) or behind (`-`) UTC (Coordinated Universal Time). For example, `-05:00` indicates US Eastern Standard Time (EST).
    * `hh` - 2-digit hour based on a 24-hour clock.
    * `mm` - 2-digit minutes.

###### The Relationship Between [RFC 3339](http://tools.ietf.org/html/rfc3339) and [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601)
[RFC 3339](http://tools.ietf.org/html/rfc3339) and [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) both provide  a standard for representing dates and times using the [Gregorian Calendar](http://en.wikipedia.org/wiki/Gregorian_calendar). Per [Wikipedia](http://en.wikipedia.org/wiki/ISO_8601) and other sources:
* RFC 3339 is a profile of (i.e., derived from, based on) ISO 8601.
* RFC 3339 requires a complete representation of date and time (only fractional seconds are optional).
* RFC 3339 differs from ISO 8601 as follows:
  * RFC 3339 deviates from ISO 8601 in allowing a zero timezone offset to be specified as "-00:00", which ISO 8601 forbids.
  * RFC 3339 only allows a period character to be used as the decimal point for fractional seconds.
  * RFC 3339 allows the "T" separator to be replaced by a space or another character, while ISO 8601 allows it to be omitted.


##### UTF-8
Each API should [use UTF-8](http://utf8everywhere.org/) when it returns a JSON response because:
* Section 3 (Encoding) of the [JSON Specification (RFC 4627)](http://www.ietf.org/rfc/rfc4627.txt) states the default encoding is UTF-8.
* UTF-8 provides a greater degree of interoperability than UTF-16 or UTF-32.

Just add a `charset` notation in the HTTP `Content-Type` response Header:

```
Content-Type: application/json; charset=utf-8
```

##### Currency and Fixed-Point Values
Monetary (i.e., Currency) information (e.g., the price of an offer (option)) should follow the [ISO 4217 - Currency Code standard](http://en.wikipedia.org/wiki/ISO_4217):
* A 3 letter Alphabetic Currency Code  (i.e., the `?currency_code` below) that consists of:
  * The first 2 letters correspond to a country code as specified by the [ISO 3166-1 alpha-2 country codes](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2).
  * Third letter corresponds to the first letter of the currency name (i.e., major currency unit) for that country.
  * For example, `USD` represents the US Dollar, and `JPY` represents the Japanese Yen.
* The amount/value of the major currency unit (see `value` below) as an integer.
* The currency exponent (see `exponent` below) assumes a base of 10. For example, USD (the United States Dollar) is equal to 100 of its minor currency unit (the "Cent"). So the USD has exponent 2 (10 to the power 2 is 100, which is the number of Cents in a Dollar). The `exponent` is the number of digits after the decimal operator.

Example:
```
{
  "price": [
    {?currency_code": "USD", "value": 1000, "exponent": 2 },
    {"currency_code": "JPY", "value": 200, "exponent": 0 }
  ]
}
```

In the above example:
* US Dollar amount is $10.00 USD, and the Japanese Yen amount is 200 JPY.
* Please note that the code `JPY` has the `exponent` 0, because its minor unit, the Sen (although nominally valued at 100th of a Yen), is of such negligible value that it is no longer used.

Fixed-point values (e.g. discount percentage) should follow a similar format:
```
{
  "tax_rate":
    {"value": 875, "exponent": 2 }
}
```

In the above example:
* The tax rate is 8.75%.


#### Third Parties and XML
Sometimes third parties (e.g., San Diego Zoo, SeaWorld/Busch Gardens) use XML, and our applications would simply consume the XML and convert it to JSON. The use of XML should be limited to data exchange with the third parties that still use XML.


### Consumer Tooling

#### Ruby Consumer Tools

Client gems are a bit of an anti-pattern that can lead to service logic
being duplicated client-side. Moving forward we'd like teams to rely on
libraries like `ls-api_client` and `ls-api_model` to integrate their
new services into applications. These libraries are fairly stable but
improvements are always welcome.


### Artifacts
Each API should come with the following artifacts to help consumers use the API:
* Documentation
* Executable Examples

#### Documentation
Each API should have human-readable documentation so that consumers
know how to use the API.

All of our APIs should use [Swagger](http://swagger.io/) to generate
documentation. Many of our APIs already use it successfully. With
Swagger, you use a JSON to specify an API's endpoints, and then
generate the documentation as HTML.

Our internal Swagger documentation lists all published swagger docs for
all implementing services.

Here are some resources for getting started with [Swagger](http://swagger.io/):
* [SOA Series Part 3: Documenting and Generating Your APIs](https://techblog.livingsocial.com/blog/2014/06/26/soa-series-part-3-documenting-and-generating-your-apis/)

#### Executable Examples
* Each API should provide executable examples that users can test directly from their terminal (e.g., `curl`),
a browser plugin (e.g., Advanced REST Client Plugin in [Chrome](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo?hl=en-US) or [Firefox](https://addons.mozilla.org/en-US/firefox/addon/restclient/)), or client GUI (e.g., [Paw](https://luckymarmot.com/paw)). This reduces dependencies between applications and simplifies testing.
* Each API (regardless of platform) should be deployed in the staging environment to support this effort.
* See also [Response Validations](#response-validations) for info on test and production validations.


### Performance
Both the API Consumer (i.e., the Client) and Producer (i.e., the Server) play a role in improving API performance.

#### Consumer-Side
The fastest API is call is one that isn't made in the first place. The API Consumer should consider caching if it can live with data that is slightly out of date. Please see [SOA Series Part 4: Caching Service Responses Client-Side](https://techblog.livingsocial.com/blog/2014/08/06/soa-series-part-4-caching-service-responses-client-side/) for details on how to implement this technique.

#### Producer-Side
There are several ways to improve the performance of an API on the Producer/Server side:
* Reduce the Response Size
  * Use [Pagination](#pagination) to manage large result sets.
  * [Condense the JSON response with HTTP Compression](#condense-json-response-with-http-compression).
* Make the API work less with a [Conditional `GET` with Caching and `ETag`](#conditional-get-with-caching-and-etag).

##### Condense JSON Response with HTTP Compression
[HTTP Compression](http://en.wikipedia.org/wiki/HTTP_compression) improves data transfer speeds and reduces bandwidth usage. There are 2 common compression schemes:
* [GZip](http://en.wikipedia.org/wiki/Gzip)
* [Deflate](http://en.wikipedia.org/wiki/DEFLATE)

We should consider leveraging [GZip](http://en.wikipedia.org/wiki/Gzip) to compress JSON Responses. Many implementations have seen a 60-80% reduction in payload size. Here are a couple of implementation suggestions:
* Clojure - Use [`java.util.zip.GZIPOutputStream`](https://docs.oracle.com/javase/7/docs/api/java/util/zip/GZIPOutputStream.html)
* Rails - Use [`Rack::Deflater`](http://www.rubydoc.info/github/rack/rack/master/Rack/Deflater). Please see [Honey, I shrunk the internet! - Content Compression via Rack::Deflater](https://robots.thoughtbot.com/content-compression-with-rack-deflater) for an example of how to use `Rack::Deflater`.

##### Conditional `GET` with Caching and `ETag`
Caching improves scalability by reducing calls to retrieve requested data, either from databases or other services. Each API should include an `ETag` HTTP Header in all responses to identify the specific version of the returned resource. The following table describes the other [HTTP Request Headers](http://en.wikipedia.org/wiki/List_of_HTTP_header_fields) that work with the `ETag` header:

| HTTP Request Header | Description                                                                                                                                            | Example                                           |
|:--------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------|
| `Cache-Control`     | The maximum number of seconds (max age) a response can be cached. However, if caching is not supported for the response, then `no-cache` is the value. | `Cache-Control: 360` or `Cache-Control: no-cache` |
| `Date`              | The date and time that the message was sent (in [RFC 1123 format](http://www.csgnetwork.com/timerfc1123calc.html)).                                    | `Date: Sun, 06 Nov 1994 08:49:37 GMT`             |
| `Pragma`            | When `Cache-Control` is set to `no-cache`, this header is also set to `no-cache`. Otherwise, it is not present.                                        | `Pragma: no-cache`                                |

The following table describes the related [HTTP Response Headers](http://en.wikipedia.org/wiki/List_of_HTTP_header_fields):


| HTTP Response Header | Description                                                                                                                                                                                                                                                                                              | Example                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|:---------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Cache-Control`      | The maximum number of seconds (max age) a response can be cached. However, if caching is not supported for the response, then `no-cache` is the value.                                                                                                                                                   | `Cache-Control: 360` `Cache-Control: no-cache`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `Date`               | The date and time that the message was sent (in [RFC 1123 format](http://www.csgnetwork.com/timerfc1123calc.html)).                                                                                                                                                                                      | `Date: Sun, 06 Nov 1994                                                                                                                                                                                                                                                               08:49:37 GMT`        |                                                                                                                                                                                                                                          |                                                                                           |
| `ETag`               | Useful for validating the freshness of cached representations, as well as helping with conditional read and update operations (`GET` and `PUT`, respectively). Its value is an arbitrary string for the version of a representation, often a Hash that represents the value of the underlying domain object. | `ETag: "686897696a7c876b7e"`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `Expires`            | If max age is given, contains the timestamp (in [RFC 1123 format](http://www.csgnetwork.com/timerfc1123calc.html)) for when the response expires, which is the value of Date (e.g. now) plus max age. If caching is not supported for the response, this header is not present.                          | `Expires: Sun, 06 Nov 1994 08:49:37 GMT`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `Last-Modified`      | The timestamp that the resource itself was                                                                                                         modified last (in [RFC 1123 format](http://www.csgnetwork.com/timerfc1123calc.html)).                                                                 | `Last-Modified: Sun, 06 Nov 1994 08:49:37 GMT`   |                                                                                                                                                                                                                                         | `Pragma`             | When `Cache-Control` is set to `no-cache`, this header is also set to `no-cache`. Otherwise, it is not present.                                                                                                                                                                                          | `Pragma: no-cache` |

Here's an example set of HTTP Response Headers in response to a GET request on a resource that enables caching for one day (24 hours):

```
Cache-Control: 86400
Date: Wed, 29 Feb 2012 23:01:10 GMT
Last-Modified: Mon, 28 Feb 2011 13:10:14 GMT
Expires: Thu, 01 Mar 2012 23:01:10 GMT
```

In the above example, the API would:
* Cache data for a resource for 24 hours. During that time:
  * The API would pull data from a cache rather than pull a new copy of the data from its original source (either a database or another service).
* Check the `Cache-Control` header vs the `Date` header in each HTTP Request to determine if the cache has expired.
* If the cache has expired:
  * Get a fresh copy of the data from the original source.
  * Hash the data and store it in the cache.
  * Generate a new `ETag` header and include it in the response.

Rather than duplicating the `Cache-Control` header vs `Date` header in each API, factor it out to a common place:
* In Rails, add a `before_filter` in `app/controllers/application_controller.rb` for your project.
* In Clojure, add a [Ring filter](https://crossclj.info/ns/ring-filter-routes/latest/ring.middleware.filter-routes.html) that runs before the impacted routes in the API.

#### Performance References
* [SOA Series Part 4: Caching Service Responses Client-Side](https://techblog.livingsocial.com/blog/2014/08/06/soa-series-part-4-caching-service-responses-client-side/)
* [SOA Series Part 6: Optimizing Your Service for Client Performance](https://techblog.livingsocial.com/blog/2014/09/30/soa-series-part-6-optimizing-your-service-for-client-performance/)
* [Support Caching with ETags - Heroku API Design Guide](https://github.com/interagent/http-api-design#support-etags-for-caching)
* See the "Caching and Scalability" and "The ETag Header" sections from the [Pearson eCollege RESTful Service Best Practices Guide](https://github.com/tfredrich/RestApiTutorial.com/raw/master/media/RESTful%20Best%20Practices-v1_2.pdf)
* [Facebook - Using ETags](https://developers.facebook.com/docs/marketing-api/etags)
* [Google Developer - Blogger JSON API: Performance Tips](https://developers.google.com/blogger/docs/3.0/performance)
* Investigate adding 'Expires', 'Cache-Control' and / or ETag headers to CMS API responses
  * [API Performance discussion on Slack](https://livingsocial.slack.com/archives/architecture/p1434123003000516) # TODO

### Testing
Here's what most developers test for:
* Make sure that fields haven't changed.
* Check expected HTTP statuses.
* Check canned responses (example offers for catalog service).

#### Ruby Test Tooling
Here's a typical Rails-based API test environment:
* BDD/TDD with [RSpec](http://rspec.info/), [MiniTest](https://github.com/seattlerb/minitest), etc.
* [CodeClimate](https://codeclimate.com/) for Test Coverage.
* [VCR](https://github.com/vcr/vcr) to record and replay API calls so that test suites run faster. See [How to Use VCR](VCR.md) for more information.
* See also [Response Validations](#response-validations)

#### Mobile Test Tooling
The Mobile teams do something similar to [VCR](https://github.com/vcr/vcr). They created [VCRURLConnection](https://github.com/dstnbrkr/VCRURLConnection), an iOS and OSX API to record and replay HTTP interactions. Any API to be consumed by a mobile app should use VCRURLConnection as part of their test environment.

In the future, the Mobile teams would like to have something similar to Mock Mode in their test suite.

#### Improve API Test Performance
A test can pull from 3 data sources when exercising an API:
* Invoke the actual service
* Pull from Cache
* Mock Mode

Consider the following techniques to improve the performance of API testing:
* Mock Mode:
  * The gem sends back mock objects rather than invoking the service.
  * Faster tests.
  * Reduces dependencies when testing.
* Caching:
  * The gem pulls from cache rather than call the service.
  * Faster tests.
  * Set from config file.

### API Developer Guides - Implementing the API Design Guide on our Platforms
This document only covers what an API should look like, but not how to implement it. To see best practices for implementing APIs that fit with this Design Guide, please refer (and contribute) to the following pages:

* Ruby on Rails API Developer Guide (_not started yet_)
* Clojure/Play API Developer Guide (_not started yet_)

#### Why Do We Have Separate API Developer Guides?
The Developer Guides are separate from the API Design Guide to group issues/concerns at the right level. The goal here is to maintain a clear focus in each document. Furthermore, we have 2 development platforms at LivingSocial, each of which has its own set of unique challenges and implementation concerns.

### Other API Design Guides
In addition to our experience, we've drawn on the the following external guides to help in the development of this API Design Guide:

* [HTTP Specification](http://tools.ietf.org/html/rfc7231) (IETF)
* [HTTP API Design Guide](https://github.com/interagent/http-api-design) ([Heroku](https://www.heroku.com/))
* [Pearson eCollege RESTful Service Best Practices Guide](https://github.com/tfredrich/RestApiTutorial.com/raw/master/media/RESTful%20Best%20Practices-v1_2.pdf)
* [API Evangelist](http://apievangelist.com/)
  * [API Evangelist API Producer Guide](https://s3.amazonaws.com/kinlane-productions/whitepapers/api-evangelist-api-design.pdf)
  * [Heroku HTTP API Design Guide](http://apievangelist.com/2014/08/22/the-heroku-http-api-design-guide/)
* [18F API Standards](https://github.com/18F/api-standards) - [18F](https://18f.gsa.gov/)
* [Apigee](http://apigee.com/)
  * [RESTful API Design 2nd Ed.](http://apigee.com/about/resources/webcasts/restful-api-design-second-edition)
  * [Web API Design](https://pages.apigee.com/web-api-design-website-h-ebook-registration.html)
  * [API Facade Pattern](https://pages.apigee.com/api-facade-pattern-ebook.html)
  * [RESTful API Design: consolidate API requests in one subdomain](https://blog.apigee.com/detail/restful_api_design_consolidate_api_requests_in_one_subdomain)
* [A Practical Approach to API Design](https://leanpub.com/restful-api-design) ([Leanpub](https://leanpub.com/))
* [Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
* [Dr. Roy Fielding's REST Dissertation](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
* [Postel's Robustness Principle](http://en.wikipedia.org/wiki/Robustness_principle)

#### JSON References
* http://www.revillweb.com/articles/why-use-json/
* https://blog.appfog.com/why-json-will-continue-to-push-xml-out-of-the-picture/
* See ["Just Use JSON" in the 18F API Standards](https://github.com/18F/api-standards#just-use-json).
