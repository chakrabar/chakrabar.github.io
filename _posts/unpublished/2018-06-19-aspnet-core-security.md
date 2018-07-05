---
layout: post
title: "ASP.NET Core security"
excerpt: "let's see..."
date: 2018-06-19
tags: [tech, aspnet, core, aspnetcore, netcore, security]
categories: articles
comments: true
share: true
published: false
---

ASP.NET Core Identity
=====================
for authentication & authorization
 - user account & password management
 - login, logout
 - external authentication providers (Google, Facebook etc.)
 - two-factor authentication
 - customization
 
Uses claim-based authentication, first introduced in .NET 4.5
 - a claim is a name-value property of an identity
 - An identity can have multiple claims
 e.g. A student [{name:John}, {dob: 12/20/1995}, {email:john@institue.com}]
 
ClaimsPrincipal has Identity[], e.g. John [Student Id Card, Driving License]
ClaimsIdentity e.g. [{name:John}, {dob: 12/20/1995}, {email:john@institue.com}]

i.e. To have a specific access (e.g. to add new books), the user has to have specific claim(s) (e.g. to have a property admin-level)

It generally works on Cookie-based authentication. On first authentication, a cookie is set. Browser saves that and sends that to each request to the domain. On server side, it validates the cookie, and based on that it sends back response. The cookie middleware in ASP.NET Core, parses the cookie and adds all claims principle to the HttpContext.

Use the "AspNetCore.Identity" to search NuGet, and use the one that is required - like SQL, Azure tables, Active Directory etc.

public void ConfigureServices(IServiceCollection services)
{
  services.AddDbContext<IdentityDbCOntext>(options => 
    options.UseSqlServer(Configuration.GetConnectionString("MyConnString"),
	optionsBuilders =>
	  optionsBuilders.MigrationsAssembly("MySolution.MyCoolProject")));
  
  services.AddIdentity<IdentityUser, IdentityRole>()
    .AddEntityFrameworkStores<IdentityDbCOntext>()
	.AddDefaultTokenProviders();
  
  services.AddMvc(); //etc.
  
  services.AddAuthorization(options =>
  {
    options.AddPolicy("FacultyOnly",
	  policy =>
	  policy.RequireClaim("FacultyNumber")) //user needs to have this property
  });
}

ASP.NET Core Token based authentication
=======================================

Basically, a token is used rather than cookie for per-request authentication.
The token may include identity & claims information. Server does not store it, it's stored by client.

WHY?
The main limitation of cookie-based authentication is that, it works only for a single domain (web) & sub-domains. Because the browser will send the specific cookie for the specific domain & sub-domains. 
When you have multiple clients, like - web, mobile, api, thick client etc. cookie cannot support this distributed architecture. On the other hand, token based authentication can work across distributed systems and varied clients.
As a result, it provides reusability & scalability. Also supports SSO (single login across multiple applications)
Finally, since token is stored at client with all relevant info, it supports stateless architecture and reduces load on server.

Security Token Service (STS)
or Authorization Server, Identity Provider

JSON Web Token (JWT)
- industry standard & platform independent
- Self-contained
  - header, payload, signature
{
  header: {
    type: "JWT", alg: "HS256"
  },
  payload: {
    exp: "1300819380", name: "John Doe", admin: true
  },
  signature: {
    HMACSHA256 //this is hashed from header & payload with the secret key
  }
}

IdentityServer is an open-source framework that can be used to create custom Token Service, in ASP.NET COre apps

The IdentityServer needs to keep a  collections of clients that can use the service
Users are stored on the server with an unique id, user-name, password & related claims
Resources are things that are protected with the identity token. They have to be registered with IdentityServer. It has two parts
 - Identity: e.g. user claims
 - APIs: protected functionalities
There are 2 types of tokens
 - Identity token - on authentication
 - Access token - when user wants to access a resource, like call an API. this is sent to the API call
 
IdentityServer implements 2 standard protocols
 - OpenID Connect : authentication protocol amd extension on top of OAuth 2.0
 - OAuth 2.0 : industry-standard protocol for authorization
IdentityServer implements them both

IdentityServer also supports external login like Google, Facebook etc.

XSS
===
Cross-site scripting attack
To run a malicious script on the page. Easiest way is to add that script as some input data (e.g. a textbox), so that it'll execute when that data is sent back to the page.

To protect, form data must be HTML encoded, query strings URL encoded etc.
In ASP.NET Core, the @ in razor pages (e.g. @User.Name) already HTML encripts data. HTML encryption can also be used as need basis. The @Html.Raw() skips this encoding.

XSRF
====
Cross-site Request Forgery
For example, you get a malicious mail that takes you to a separate website, and makes you click a button (or does that automatically through a script).
Assuming you are already logged in to a secure site from the same browser, the browser will send the valid authentication cookie. This request will succeed and can do harmful things.

ASP.NET Core can use some NuGet packages to add anti-forgery keys to the page. This basically adds some random encrypted value in the cookie, as well as in some hidden form field. the form fileds request token also includes some additional user specific data. On post to server, it matches those two values. If they are not same, or request token does not match a valid user, the server rejects the request.

SQL Injection
=============
Injecting malicious SQL scripts into the application. Look at the following sample query

Sample dynamic query with string concatination on user input:
"SELECT * FROM Students WHERE LastName = '" + lastName + "'"

Possible result:
Entered user input: Smith' OR '1'='1'--
SELECT * FROM Students WHERE LastName = 'Smith' OR '1'='1'--'; --get data for all students

Possible result 2:
Entered user input: Smith';DROP table Students;--
SELECT * FROM Students WHERE LastName = 'Smith';DROP table Students;--'; --DROP Students table!!

How to protect:
- Try to avoid dynamic queries
- Use good ORM like EF
- For dynamic queries or SPs, use parameterized queries

Parameterized queries automatically sanitize query parameters. 
The parameters can also be sanitized manually to remove unsafe characters like "'"
Also, use a database account with least priviledge, that does not have rights to create or drop tables etc.

CORS
====
Cross-Origin Resource Sharing
Browsers apply same-origin policy by default: If applications need to share data with another site, they must have same origins. That means
They should have same domian, scheme, hosts & port
CORS allows server to share data with resources with trusted origin.

public void ConfigureServices(IServiceCollection services)
{
  services.AddCors(options => 
  {
    options.AddPolicy("SomePolicyName",
	  builder => 
	    builder.WithOrigins("http://myothersite.com")
		.WithMethods("GET")
		.AllowAnyHeader());
  });
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
  app.UseCors("SomePolicyName"); //registers the policy globally in the pipeline
  app.UseMvc();
}

Open Redirect Attack
====================
You are asked to login to a trusted site, but because of the malicious url, it tries to take you to an untrusted site.
http://mysite.com/Account/Login?url=http://mysite2.com
Then maybe, it'll ask you to reenter your credentials. Thinking you mistyped it last time, you'll enter them again. Now that site has your credentials!

In login method, one can check if the redirect url is a local one

//after authentication
if (Url.IsLocalUrl(returnUrl))
  return Redirect(returnUrl);
else
  return Redirect(Some local url);
  
URL Manipulation
================
Simply change URl manually to probe a site
http://mysite.com/getresults?id=110&isAdmin=false
Anyone can change these values to get other user or even admin data.

URL are always public. Do NOT put sensitive data there.
If needed, put them in encrypted form
Also implement access control
GET requests should be idempotent, meaning calling them over and over should get same results. In simple words, they should not make any change on server side

ENCRYPTION
==========
Encryption algorithms change a simple text into cyphertext using a specific key. The algorithms are of two types
- Symmetric: Same key is used for encryption & decryption. So the key needs to be sent to client, so it can get the data, e.g. AES256 (Advanced Encryption Standard)
- Asymmetric: or the public-private key cryptography. This key-pair is generated together. The public key is distributed for clients to encrypt data. Only server has the private key that can decrypt the data.

Asymmetric encryption relies on complex calculations and is generally slower. Symmetric encryption works better for large data.

In hybrid encryption, the symmetric key is encrypted with public key cryptography. From https://en.wikipedia.org/wiki/Hybrid_cryptosystem

To encrypt a message addressed to Alice in a hybrid cryptosystem, Bob does the following:

- Obtains Alice's public key.
- Generates a fresh symmetric key for the data encapsulation scheme.
- Encrypts the message under the data encapsulation scheme, using the symmetric key just generated.
- Encrypt the symmetric key under the key encapsulation scheme, using Alice's public key.
- Send both of these encryptions to Alice

To decrypt this hybrid ciphertext, Alice does the following:

- Uses her private key to decrypt the symmetric key contained in the key encapsulation segment.
- Uses this symmetric key to decrypt the message contained in the data encapsulation segment.

Hashing: Creats an encrypted text from a plain text. This process is irreversible. The hashed value cannot be un-hashed. But the same value will always produce the same hash, and even the most minor change in value will create a different hash. hashing is frequently used in passords & certificates.

Hashing algorithms like MD5 & SHA1 are considered weak. Use a stronger hashing like SHA256.
