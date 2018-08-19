---
layout: post
title: "Web Security with ASP.NET Core"
excerpt: "let's see..."
date: 2018-06-19
tags: [tech, aspnet, core, aspnetcore, netcore, security]
categories: articles
comments: true
share: true
published: false
---

## General Web Security

The very basic of web security is to protect the communication between the client and the server, so that no one else can read or manipulate the information exchange.

* SSL : Secure Socket Layer, latest was 3.0
* TLS: Built on top of SSL, current versions are `1.1` and `1.2`
* HTTPS: Secure HTTP over SSL/TLS channel

Since SSL is older, and there were some vulnerabilities found in SSL, **TLS is mostly used these days than SSL**. But the terms SSL and TLS are used interchangeably. In some ways, conceptually, TLS can be thought as newer version of SSL.

SSL certificates are issues by established <u>Certificate Authorities (CA)</u> like - DigiCert, Symantec/VeriSign, Entrust, Let's Encrypt etc.

It is important to note that, certificates are not bound to a specific version of protocol like SSL/TLS, the same certificate can be used over either protocol.

#### Basic SSL handshake

* Clients connects to server and asks to verify itself
* Server does that by sending back a SSL certificate
* If client trusts the CA, it proceeds. Else the connection does not establish
* Client then creates a `symmetric session key`, uses the `asymmetric public key` from server (sent with the certificate) to encrypt it and sends it back to the server
* Server decrypts the key (hence verifies the client), and keeps the key to be used for the session
* Server sends back an acknowledgment encrypted with the session key
* Client receives & decrypts the acknowledgment and the connection is established
* Going forward, the session key is used to encrypt all the communication 

**Note:** More information on symmetric & asymmetric encryption later in the encryption section.

#### What is encrypted in a HTTPS communication?

TLS/SSL stands between TCP and HTTP. So, once the channel is established, everything that goes to HTTPS is encrypted. That includes the URI itself, including the query parameters, the HTTP method, the headers and the payload data! That means, everything is encrypted.

But remember 2 things

1. Just the domain name part (e.g. www.example.com) is still readable as that is used to identify the server during DNS lookup and routing
2. Even encrypted, it's not totally safe to have secure data in URI, as
   1. It is readable in browser address-bar
   2. They're saved in browser history and logs
   3. When navigating on the web, the current page gets the previous page URI as referrer link

## ASP.NET Core Identity

The ASP.NET Core Identity is the module/package that handles all that is related to user identity, authentication & authorization. It comes with following features for authentication & authorization

* User account & password management
* Login, logout
* External authentication providers (Google, Facebook etc.) through OAuth
* Two-factor authentication
* And customization

It uses **claim-based authentication**, first introduced in .NET 4.5

* A claim is a name-value property of an identity
* An identity can have multiple claims
 e.g. A student [{name: "John"}, {dob: "12/20/1995"}, {email: "john@institue.com"}]

ClaimsPrincipal has `Identity[]`, e.g. John can have ClaimsPrincipal that has identities like [Student Id Card, Driving License].
Where a claimsIdentity can be student ID card e.g. [{name: "John"}, {studentId: "CS12032"} {dob: "12/20/1995"}, {email: "john@institue.com"}]

i.e. To have a specific access (e.g. to add new books to library system), the user has to have specific claim(s) (e.g. to have a property library-admin)

### Cookie based claims identity

It generally works on Cookie-based authentication. On first authentication, a cookie is set.

The cookie contains the identity and related claims in encrypted format. Browser saves that and sends that with each request to the domain.

**Note:** Understand that is is pretty different from _traditional cookie based authentication_. There, the server maintains all user related state information (like user details and permissions etc.) in server-side session. The session-ID is put in the cookie. Every time the client sends a request, it sends the cookie. On the server side, the user details are retrieved from session with the session-ID from the cookie. When user logs out, server removes the session info and the cookie in invalidated on client as well.
{: .notice--info}

On server side, it validates the cookie, and based on that it verifies the user and claim based permissions. The cookie middleware in ASP.NET Core, parses the cookie and adds all claims principle to the HttpContext.

Use the "AspNetCore.Identity" to search NuGet, and use the one that is required - like SQL, Azure tables, Active Directory etc.

```cs
//Startup
public void ConfigureServices(IServiceCollection services)
{
  services.AddDbContext<IdentityDbCOntext>(options => 
    options.UseSqlServer(Configuration.GetConnectionString("MyConnString"),
	optionsBuilders =>
	  optionsBuilders.MigrationsAssembly("MySolution.MyCoolProject"))); //so that the project finds DbContext easily
  
  services.AddIdentity<IdentityUser, IdentityRole>()
    .AddEntityFrameworkStores<IdentityDbCOntext>()
	.AddDefaultTokenProviders();
  
  services.AddMvc(); //etc.
  
  services.AddAuthorization(options => //adding authorization policy
  {
    options.AddPolicy("FacultyOnly", //policy name, to be used for authorization
	  policy =>
	  policy.RequireClaim("FacultyNumber")) //user needs to have this property
  });
}

public void Configure(IAppBuilder app, ...)
{
  app.UseStaticFiles();
  app.UseIdentity(); //use cookie based identity system
  app.UseMvc();
}

//Account controller
//for valid faculty, add the FacultyNumber claim
public IActionResult Register(RegisterViewModel model)
{
  IdentityUser user = new IdentityUser { Email = model.Email, Name = model.Name };
  if (!string.IsNullOrEmpty(model.FacultyNumber))
  {
    user.Claims.Add(new IdentityUserClaim<string>
      {
        ClaimType = "FacultyNumber", ClaimValue = model.FacultyNumber
      });
  }
}
```

<u>In ASP.NET Core</u>, to

1. Enforce **HTTPS**, add the `UseHttps()` filter globally, or to specific controllers or action. To enforce it for static files, add a `ReWrite` rule to redirect HTTP requests to HTTPS.
2. Enforce **authentication**, use `[Authorize]` filter or required controllers and actions. To specifically allow non-authenticated use on a controller with [Authorize], use `[AllowAnonymous]` filter.
3. To have actual **authorization**, use the in-built policy based authentication. For that, first add custom policies in `ConfigureServices()` method as shown above. Then to required controller action, add [Authorize(Policy = "FacultyOnly")]

### ASP.NET Core Token based authentication

Basically, a token is used rather than cookie for per-request authentication. The token may include identity & claims information. The token includes a signature by the server as part of itself. Then the whole thing is encrypted (generally base64 encryption) and sent back to client.

WHY?

The main limitation of cookie-based authentication is that, it works only for a single domain (web) or sub-domain. Because the browser will send the specific cookie for the specific domain/sub-domain. Also the same cookie is hard to share between multiple clients.

The main differences for token based authentication are

1. **Tokens are stateless on server**. All the information are encrypted and signed, and written on token that is given back to client to be used for each request. Server does not store any data, it fetches back the required info from the token every time.
2. **Tokens are generally passed around in (authorization) header**. So it does not have the issues of single client and single domain. The client can request related data from another sub domain or service with the same token (think smart client like a mobile app), and it'll work if the other systems knows how to handle the token. Also, each request can go to any server or cluster node within the same back-end system. But, tokens can also be transmitted through URI or post data. Or, cookie as well, if the need be.

* When you have multiple clients, like - web, mobile, api, thick client etc. Cookie cannot support this distributed architecture. On the other hand, token based authentication can work across distributed systems and varied clients.
* As a result, token provides re-usability & scalability. Also supports SSO (single login across multiple applications) and OAuth.
* Finally, since token is stored at client with all relevant info, it supports stateless architecture and reduces load on server.

In ASP.NET COre, this is taken care by the Security Token Service (STS) - or Authorization Server, Identity Provider

#### JSON Web Token

JWT in short, also called "jaw-t". This is the most common format of tokens used these days.

* Industry standard & platform independent
* Self-contained with header, payload (the claims), signature

```javascript
{
  header: {
    type: "JWT", alg: "SHA256"
  },
  payload: {
    exp: "1300819380", name: "John Doe", admin: true
  },
  signature: {
    HMACSHA256 //this is hashed from header & payload with the secret key
  }
}
```

The signature is hashed from the token data, with the server's private key. Server verifies the token by hashing the token data again and matching with the signature. If anything has been tampered with, that is caught.

#### IdentityServer

IdentityServer is an open-source framework that can be used to create custom Token Service, in ASP.NET Core apps.

The IdentityServer needs to keep a  collections of clients that can use the service. Users are stored on the server with an unique id, user-name, password & related claims. Resources are things that are protected with the identity token. They have to be registered with IdentityServer.

It has two parts

* Identity: e.g. user claims
* APIs: protected functionalities

There are 2 types of tokens

* Identity token - on authentication
* Access token - when user wants to access a resource, like call an API. this is sent to the API call

IdentityServer implements 2 standard protocols

* **OpenID Connect** : authentication protocol amd extension on top of OAuth 2.0
* **OAuth 2.0** : industry-standard protocol for authorization

IdentityServer also supports external login like Google, Facebook, Twitter etc.

## Common types of attacks

#### XSS : Cross-Site Scripting

To run a malicious script on the page. Easiest way is to add that script as some input data (e.g. a textbox), so that it'll execute when that data is sent back to the page.

To protect, form data must be HTML encoded, query strings URL encoded etc.
In ASP.NET Core, the @ in razor pages (e.g. @User.Name) already HTML encripts data. HTML encryption can also be used as need basis. The @Html.Raw() skips this encoding.

#### XSRF : Cross-Site Request Forgery

For example, you get a malicious mail that takes you to a unintended website (which may look similar to your known site, like your XYZ bank site).

And then it makes you click a button (or does that automatically through a script), which posts a request to the other site (the actual XYZ bank site). Assuming you are already logged in to a secure site from the same browser, the browser will send the valid authentication cookie. This request will succeed and can do harmful things.

ASP.NET Core can use some NuGet packages to add **anti-forgery token** to the page. This basically adds some random encrypted value in the cookie, as well as in some hidden form field. the form field's request token also includes some additional user specific data. On post to server, it matches those two values. If they are not same (the random key in cookie and from form post), or request token does not match a valid user, the server rejects the request.

#### SQL Injection

Injecting malicious SQL scripts into the application. Look at the following sample query

Sample dynamic query with string concatination on user input:
"SELECT * FROM Students WHERE LastName = '" + text_lastName.Value + "'"

Possible attack with intentional query input:

1. Get all user data. Entered user input => Smith' OR '1'='1'--
SELECT * FROM Students WHERE LastName = 'Smith' OR '1'='1'--';
2. Drop the student table with data! Entered user input => Smith';DROP table Students;--
SELECT * FROM Students WHERE LastName = 'Smith';DROP table Students;--';

How to protect:

* Try to avoid dynamic queries
* Use good ORM like EF
* For dynamic queries or SPs, use parameterized queries

Parameterized queries automatically sanitize query parameters. The parameters can also be sanitized manually to remove unsafe characters like "'". Also, use a database account with least privilege, that does not have rights to create or drop tables etc.

#### Open Redirect Attack

You are asked to login to a trusted site, but because of the malicious URL, it tries to take you to an untrusted site.

For example, you are simply taken to your actual XYZ bank site. But that URL has a redirect to another fake XYZ bank site, which looks exactly same as the original site. Most people will not notive that malicious redirect link in the URL, e.g. http://xyzbank.com/Account/Login?url=http://xysbank.com. Once you have successfully logged in, it sends you to the fake site which will have 'incorrect credentials' page open. Then maybe, it'll ask you to reenter your credentials. Thinking you mistyped it last time, you'll enter them again. Now that fake site has your credentials!

To prevent this, in login method, one can check if the redirect URL is a local one

```cs
//after authentication
if (Url.IsLocalUrl(returnUrl))
  return Redirect(returnUrl);
else
  return Redirect(Some local url);
```
  
#### URL Manipulation / URL Probing

Simply change URl manually to probe a site
http://mysite.com/getresults?id=110&isAdmin=false
Anyone can change these values to get other user or even admin data.

URL are always public. Do NOT put sensitive data there. If needed, put them in encrypted form. Also implement access control.

<u>GET requests should be idempotent</u>, meaning calling them over and over should get same results. In simple words, they should not make any change on server side.

#### CORS

Cross-Origin Resource Sharing (<u>Not an attack</u>)

Browsers apply same-origin policy by default: If applications need to share data with another site, they must have same origins. That means - they should have same domain, scheme, hosts & port. CORS allows server to share data or resources with trusted origin(s).

```cs
public void ConfigureServices(IServiceCollection services)
{
  services.AddCors(options =>
  {
    options.AddPolicy("SomePolicyName",
      builder =>
        builder
        .WithOrigins("http://myothersite.com")
        .WithMethods("GET")
        .AllowAnyHeader());
  });
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
  app.UseCors("SomePolicyName"); //registers the policy globally in the pipeline
  app.UseMvc();
}
```

## ENCRYPTION

Encryption algorithms change a simple text into **ciphertext** using a specific key. The algorithms are of two types

* **Symmetric**: Same key is used for encryption & decryption. So the key needs to be sent to client, so it can read the data, e.g. AES256 (Advanced Encryption Standard)
* **Asymmetric**: Or the public-private key cryptography. This key-pair is generated together. The public key is distributed for clients to encrypt data. Only server has the private key that can decrypt the data.

Asymmetric encryption relies on complex calculations and is generally slower. Symmetric encryption works better for large data, and they are faster.

In **Hybrid Encryption**, the symmetric key is encrypted with public key cryptography. This is the general strategy in today's HTTPS communication. From [Wikipedia](https://en.wikipedia.org/wiki/Hybrid_cryptosystem) -

To encrypt a message addressed to Alice in a hybrid crypto system, Bob does the following:

* Obtains Alice's public key.
* Generates a fresh symmetric key for the data encapsulation scheme.
* Encrypts the message under the data encapsulation scheme, using the symmetric key just generated.
* Encrypt the symmetric key under the key encapsulation scheme, using Alice's public key.
* Send both of these encryptions to Alice.

To decrypt this hybrid ciphertext, Alice does the following:

* Uses her private key to decrypt the symmetric key contained in the key encapsulation segment.
* Uses this symmetric key to decrypt the message contained in the data encapsulation segment.

#### Hashing 

Creates an _"encrypted"_ text from a plain text. <u>This process is irreversible</u>. The hashed value cannot be un-hashed. But the same value will always produce the same hash with the same hashing key. And even the most minor change in value will create a different hash. hashing is frequently used in passwords & certificates.

Hashing algorithms like MD5 & SHA1 are considered weak. Modern systems use a stronger hashing like SHA256.

#### Data Protection

ASP.NET Core data protection system is new and replaces cryptographic stack in ASP.NET 1.x - 4.x. This is used by cookie middleware and anti-forgery validation. Can be setup and used to protect any data, and comes with easy to use APIs.

Uses <u>AES256 for symmetric and SHA256 for asymmetric encryption</u>. Has built-in validation and key management. Encryption keys are rotated (auto-re-generated) every 90 days by default. Old keys are still kept though, to enable decryption.