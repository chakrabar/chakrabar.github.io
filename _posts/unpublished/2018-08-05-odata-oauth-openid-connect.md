---
layout: post
title: "OData, OAuth & OpenID Connect"
excerpt: "Basics concept and code samples for closures"
date: 2018-08-05
tags: [closure, javascript, C#]
categories: articles
comments: true
share: true
published: false
---

# OData, OAuth, OpenID Connect

1. https://www.linkedin.com/learning/programming-foundations-web-services
2. https://www.linkedin.com/learning/web-security-oauth-and-openid-connect

## OData

OData is a standard (and protocol) for creating web services. It's a combination of architectural style and messaging format. The aim is to simplify web services with standard protocols & messaging formats. Some of the main features are

* Built on REST standards
* Requests are URI based rather than JSON/XML payload
* Response is in a standard format for all services. It's generally XML, but can be JSON as well. Till version 3, the the default format was the **ATOM format**, a RSS like XML structure. Since [version 4.0](http://docs.oasis-open.org/odata/odata-json-format/v4.0/odata-json-format-v4.0.html) it has been less verbose JSON format
* Can work with HTTP verbs like GET, POST, PUT, DELETE
* The metadata schema is part of the response. So a client can programatically find out about the data structure and related links (similar to HATEOS standard in REST). For example, the `$metadata` for the service https://services.odata.org/OData/OData.svc is available at https://services.odata.org/OData/OData.svc/$metadata
* There are standard query parameters, which supports standard query like `$top`, `$skip`, `$filter`, `$count`, `$orderby` etc.

#### Servers & Clients

Clients are application/packages that can read OData response and transform them back to native data types (e.g. C# objects), and make new requests. Servers help format URIs and handle requests. See more [here](https://www.odata.org/libraries/).

A sample URI/URL in OData would look like (this is an actual service)

https://services.odata.org/OData/OData.svc/Products?$top=2&$orderby=Price

Note:

1. The URI, mainly resource path (/OData/OData.svc), data collection (Products) & parameters are _generally_ case sensitive
2. The system parameters (standard parameters defined by OData) are prefixed with `$`, like `$top` and `$orderby`
3. The standard response have content of type `Content-Type: application/atom+xml;type=feed;charset=utf-8`

More examples

* Another sample URI https://services.odata.org/odata/odata.svc/Products?$filter=Price%20lt%2010. Here we are passing a filter criteria `Price lt 10` i.e. Price < 10
* the standard ATOM response also provides linked resource references like https://services.odata.org/OData/OData.svc/Products(0)/Supplier, https://services.odata.org/OData/OData.svc/Products(0)/Categories etc.

#### Data format

The basic format, on a high level, looks like following. The content is XML by default.

```bash
feed
  id
  title
  lastUpdated
  entries[]
    entry
      id
      category
      links[] - related data
      content - the actual data, which can have any names & types
        properties - as required by specific data, e.g.
          name
          age
          gender
          etc.
```

To request data in JSON format, just add the query parameter **`$format=json`**. The returned JSON data is also much more compact. Or simply add header `Accept:application/json`

For example, following is a sample data as JSON, from https://services.odata.org/OData/OData.svc/Products?$top=2&$orderby=Price&$format=json

```javascript
{
	odata.metadata: "https://services.odata.org/OData/OData.svc/$metadata#Products",
	value: [
		{
			odata.type: "ODataDemo.FeaturedProduct",
			ID: 9,
			Name: "Lemonade",
			Description: "Classic, refreshing lemonade (Single bottle)",
			ReleaseDate: "1970-01-01T00:00:00",
			DiscontinuedDate: null,
			Rating: 7,
			Price: 1.01
		},
		{
			ID: 0,
			Name: "Bread",
			Description: "Whole grain bread",
			ReleaseDate: "1992-01-01T00:00:00",
			DiscontinuedDate: null,
			Rating: 4,
			Price: 2.5
		}
	]
}
```

**Note:** One thing to understand is, if the benefits of OData like standard format, pre-defined query param, discoverable metadata are not required, then the downside of OData is, the ATOM format adds lot of additional info to the request, making it much heavier than the REST counterpart. So, in cases like that, probably REST would serve the purpose better.
{: .notice--info}

## OAuth

* Authentication => AuthN => Who are you? => generally accomplished through login
* Authorization => AuthZ => What you're allowed to do? => follows the process of authentication

OAuth is actually an **authorization process**. It says "I don't know exactly who you are, but the 3rd party tells me what you can do!"

#### How does OAuth work?

On surface level, basic OAuth workflow is like this. Imagine you want to access a website or mobile app, which needs to verify you through a standard 3rd party login like Facebook, Google, LinkedIn or Twitter. Let's say you choose Twitter.

* The site/app sends you to the Twitter login page
* You login to authenticate yourself
* Then Twitter tells you the site/app wants to access few things about you profile (like Name & Email), and asks for your permission
* You grant the permissions by selecting the options
* Twitter sends back a token to the site/app with the authorization info
* With that token, the site/app can continue to do stuffs that you had allowed earlier
* Remember your **password is never shared** with the site/app
* You can simply revoke the permissions at any time you want

Basic pieces in OAuth

1. The request to access token flow is called `flow` or `grant type`
2. The permissions are called `scopes`, and they are custom defined by each OAuth provider
3. Access is granted via encrypted access `tokens`
4. There's no specification for payloads, but generally JWT (JSON Web Tokens) are used

#### Some standard AuthN + AuthZ approaches for web 

1. Token based API keys - a token is passed around in URI/header
   1. Simple to use
   2. Hard to rotate, also gives global-all permissions generally (well, I'd rather say it depends on the implementation)
2. Identifier + secret approach - not the same account login credentials
   1. Very similar to above
   2. More issues if details are compromised
3. OAuth
   1. App specific, well managed, subset of permissions
   2. Easy for user to revoke access
   3. Very popular and well supported in different tech stacks
   4. The main problem is, it is often misunderstood. It's not perfect and has known limitations, and there are many extensions to fill the gaps. But if used incorrectly, the problems doesn't resolve
  
#### OpenID Connect (OIDC)

A simple <u>identity extension on top of OAuth 2.0</u>, and it defines specific structure for sharing profile information such as - phone number, email etc. Designed mainly **for user and authentication**.

OIDC revolves around **scopes & claims**.

OAuth does not specify much about what goes in where, in what format. OpenID helps in that, as it's standards are more defined. The most common OAuth implementations like in the integration of Facebook etc., is generally implemented with OIDC.

#### Tokens

Tokens may or may not have any data included. Tokens are of 2 types

  1. Access token - used to access stuffs
  2. Refresh - to get a refreshed/renewed token
  
In standard **JWT** format, some of the common fields are - issuer, subject (user), audience (the site/app), expiration etc. (they are basically claims, as defined below)

Like any JWT, it'll have a signature which is a hash of the whole data. The issuer can detect if anything has been tampered with.

Stuffs related to token (not embedded in it)

1. **Scope** - the type of permissions, defined by the issuer e.g. delete repo scope in GitHub. Scopes define what site/app can on behalf of the user
2. **Claims** - set of key-value pairs within the token, information about the user, for the app (e.g. email:blob@mail.com, dp:uri-to-dp). This is generally embedded in the token