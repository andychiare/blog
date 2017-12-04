---
layout: post
title: "Using JSON Web Tokens with .NET Core 2"
description: "A practical tutorial showing how to use JWTs in a .NET Core 2 application"
date: 2017-xx-xx x:xx
category: Technical Guide, Microsoft, ASP Net Core
author:
  name: "Andrea Chiarelli"
  url: "https://twitter.com/andychiare"
  mail: "andrea.chiarelli.ac@gmail.com"
  avatar: ""
design:
  bg_color: "#3A1C5D"
  image: https://cdn.auth0.com/blog/asp-net-core-tutorial/logo.png
tags:
- .net-core
- asp.net-core
- asp.net
- c#
- oauth
- openid-connect
related:
- 2016-06-27-auth0-support-for-aspnet-core
- 2016-06-03-add-auth-to-native-desktop-csharp-apps-with-jwt
- 2017-06-05-asp-dot-net-core-authentication-tutorial.markdown
---

**TL;DR:** Unlike the previous version, .NET Core 2 provides native support to JSON Web Tokens. This allows us to integrate this technology in ASP.NET applications in an easier way. In this article, we will take a look at how to enable JWTs when creating a Web API application based on .NET Core 2.

## A quick introduction to JWTs

*JSON Web Tokens*, often shortened with JWTs, are gathering more and more popularity in the Web environment. It is an [open standard](https://tools.ietf.org/html/rfc7519) that allows transmitting data between parties as a JSON object in a compact and secure way. They are usually used in authentication and information exchange scenarios, since the data transmitted between a source and a target are digitally signed so that they can be easily verified and trusted.

The JWTs are structured in three sections:

- the *Header*
  this is a JSON object containing meta-information about the type of JWT and hash algorithm used to encrypt the data
- the *Payload*
  even this is a JSON object containing the actual data shared between source and target; these data are coded in *claims*, that is statements about an entity, typically the user
- the *Signature*
  this section allows to verify the integrity of the data, since it represents a digital signature based on the previous two sections

The three sections of a JWT are combined together into a sequence of Base64 strings separated by dots so that the data can be easily sent around in HTTP-based environments. When used in authentication, JWT technology allows a client to store session data on its side and to provide the token to the server whenever it tries to access a protected resource. Usually, the token is sent to the server in an *Authorization* HTTP header using the *Bearer* schema, and it should contain all the information that allows to grant or deny access to the resource.

Of course, this is a very quick overview of JWT, just to have a common terminology and a basic idea of what this technology is. You can find more information in [Introduction to JSON Web Tokens](https://jwt.io/introduction/).

{% include tweet_quote.html quote_text="JSON Web Tokens are a compact and self-contained way for securely transmitting information between parties as a JSON object." %}



## Configuring JWT support in .NET Core 2

Let's take a look at how to set up a .NET Core 2 application with JWT support by creating a Web API application. You can create it by using Visual Studio or via command line. In the first case you should choose the *ASP.NET Core Web Application* project template, as shown in the following picture:

![](./xxxx-xx-xx-using-jwt-with-dotnet-core-2_img/VS_ASPNET_dialog.png)

Then you need to select the type of ASP.NET application, that in our case will be *Web API*, how we can see in the following picture:

![](./xxxx-xx-xx-using-jwt-with-dotnet-core-2_img/VS_WebAPI_dalog.png)

For simplicity, we have not enabled any type of authentication since we want to focus on JWT management.

If you prefer to create your application from command line, type the following command in a command window:

```shell
dotnet new webapi -n JWT
```

This will create an ASP.NET Web API project named JWT in the current folder.

Regardless the way you have created your project, you will get in the folder the files defining the classes to setup a basic Web API application.

First of all, we change the body of `ConfigureServices` method in `Startup.cs` in order to configure support for JWT-based authentication. The following is the resulting implementation of `ConfigureServices`:

```c#
public void ConfigureServices(IServiceCollection services)
{
 services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
	.AddJwtBearer(options =>
	{
		options.TokenValidationParameters = new TokenValidationParameters
		{
			ValidateIssuer = true,
             ValidateAudience = true,
             ValidateLifetime = true,
             ValidateIssuerSigningKey = true,
             ValidIssuer = Configuration["Jwt:Issuer"],
             ValidAudience = Configuration["Jwt:Issuer"],
             IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(Configuration["Jwt:Key"]))
		};
	});

	services.AddMvc();
}
```
Here we register JWT authentication schema by using `AddAuthetication` method and specifying `JwtBearerDefaults.AuthenticationScheme`. Then we configure the authentication schema with options for JWT bearer. In particular, we specify which parameters must be taken into account in order to consider valid a JSON Web Token. Our code is saying that to consider a token valid we must validate the server that created that token (`ValidateIssuer`), we must ensure that the recipient of the token is authorized to receive it (`ValidateAudience`), we must check that the token is not expired and that the signing key of the issuer is valid (`ValidateIssuerSigningKey`). In addition, we specify the values for the issuer, the audience and the signing key. These values are stored in the `appsettings.json` file and then accessible via `Configuration` object:

```json
//appsettings.json
{
...
,
  "Jwt": {
    "Key": "veryVerySecretKey",
    "Issuer": "http://localhost:63939/"
  }
}
```

This step configures the JWT-based authentication service. In order to make the authentication service available to the application, we need to add `app.UseAuthentication()` invocation in `Configure` method: 

```c#
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
	if (env.IsDevelopment())
	{
		app.UseDeveloperExceptionPage();
	}
  
	app.UseAuthentication();

	app.UseMvc();
}
```
This change completes the configuration of our application to support JWT-based authentication.

You can find the source code of the project we are going to illustrate in this article in this [Github repository](https://github.com/andychiare/netcore2-jwt).

## Differences from previous .NET Core version

If you already knew [how .NET Core 1.x supported JWT](https://auth0.com/blog/asp-dot-net-core-authentication-tutorial/), you can find that it has been made easier.

First of all, in previous version of .NET Core you needed to install a few external packages. Now it is no longer needed since JSON Web Tokens are now natively supported.

In addition, the configuration steps have been simplified as a consequence of the overall authentication system. In fact, while in .NET Core 1.x we had a middleware for each authentication schema we would support, .NET Core 2.0 uses a single middleware handing all authentication and each authentication schema is registered as a service.

This allows to create a more compact and cleaner code.

## Unauthorized access to APIs

Once we have enabled JWT-based authentication, let's create a simple Web API. It will return a list of books when invoked with HTTP GET:

```c#
[Route("api/[controller]")]
public class BooksController : Controller
{
	[HttpGet, Authorize]
	public IEnumerable<Book> Get()
	{
		var resultBookList = new Book[] {
			new Book { Author = "Ray Bradbury",Title = "Fahrenheit 451"},
			new Book { Author = "Gabriel García Márquez", Title = "One Hundred years of Solitude"},
			new Book { Author = "George Orwell", Title = "1984"},
			new Book { Author = "Anais Nin", Title = "Delta of Venus"}
		};

		return resultBookList;
	}

	public class Book
	{
		public string Author { get; set; }
		public string Title { get; set; }
	}
}

```

As we can see, the API simply returns an array of book objects. We can notice, however, that we marked the API with the `Authorize` attribute. It will trigger the validation check of the token passed with the HTTP request.

If we run the application and make a GET request to the `/api/books` endpoint, we will get a `401` HTTP status code as a response. You can try it by running the `UnAuthorizedAccess` test in the `Test` project attached to the [project's source code](https://github.com/andychiare/netcore2-jwt) or by using a generic HTTP client such as [curl](https://curl.haxx.se/) or [Postman](https://www.getpostman.com/).

For example, by calling the API with *Postman* we will get the following result:

![](./xxxx-xx-xx-using-jwt-with-dotnet-core-2_img/401_Postman.png)

Of course, this result is due to the lack of the token, so that the access to the API has been denied.

## Creating JWT on authentication

Let's add an authentication API to our application, so that when a user is authenticated a new JWT is created and returned to the client. The following is the code that implements this API:

```c#
[Route("api/[controller]")]
public class TokenController : Controller
{
...
	[AllowAnonymous]
	[HttpPost]
	public IActionResult CreateToken([FromBody]LoginModel login)
	{
		IActionResult response = Unauthorized();
		var user = Authenticate(login);

		if (user != null)
		{
			var tokenString = BuildToken(user);
			response = Ok(new { token = tokenString });
		}

		return response;
	}

	private string BuildToken(UserModel user)
	{
		var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
		var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

		var token = new JwtSecurityToken(_config["Jwt:Issuer"],
		  _config["Jwt:Issuer"],
		  expires: DateTime.Now.AddMinutes(30),
		  signingCredentials: creds);

		return new JwtSecurityTokenHandler().WriteToken(token);
	}

	private UserModel Authenticate(LoginModel login)
	{
		UserModel user = null;

		if (login.Username == "mario" && login.Password == "secret")
		{
			user = new UserModel { Name = "Mario Rossi", Email = "mario.rossi@domain.com"};
		}
		return user;
	}
...
}
```

We omitted some code for the sake of simplicity, but remember that you can download the complete code from the [Github repository](https://github.com/andychiare/netcore2-jwt).

The first thing to notice is the presence of `AllowAnonymous` attribute. This is very important, since this must be a public API, that is an API that anyone can access to get a new token after providing his credentials.

The API responds to an HTTP POST request and expects an object containing username and password (a `LoginModel` object).

The `Authenticate` method verifies that the provided username and password are the expected ones and returns a `UserModel` object representing the user. Of course, this is a trivial implementation of the authentication process. A production-ready implementation should be more accurate as all we know.

If the `Authentication` method returns a user, that is the provided credentials are valid, the API generates a new token via the `BuildToken` method. And this is the most interesting part: here we create a JSON Web Token by using the `JwtSecurityToken` class. We pass a few parameters to the class constructor, such as the issuer, the audience (in our case are the same), the expiration date and time and the signature. Finally, the `BuildToken` method returns the token as a string, by converting it through the `WriteToken` method of the `JwtSecurityTokenHandler` class.



## Authorized access to APIs

Now we can test the two APIs we created.

First, let's get a JWT by making an HTTP POST request to `/api/token` endpoint and passing the following JSON in the request body:

```json
{"username": "mario", "password": "secret"}
```

As a response we will obtain a JSON like the following:

```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJNYXJpbyBSb3NzaSIsImVtYWlsIjoibWFyaW8ucm9zc2lAZG9tYWluLmNvbSIsImJpcnRoZGF0ZSI6IjE5ODMtMDktMjMiLCJqdGkiOiJmZjQ0YmVjOC03ZDBkLTQ3ZTEtOWJjZC03MTY4NmQ5Nzk3NzkiLCJleHAiOjE1MTIzMjIxNjgsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6NjM5MzkvIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo2MzkzOS8ifQ.9qyvnhDna3gEiGcd_ngsXZisciNOy55RjBP4ENSGfYI"
}
```

If we look at the value of the token, we will notice the three parts separated by a dot, as discussed at the beginning of this article.

Now we will try again to request the list of books, as in the previous section. However, this time we will provide the token as an authentication HTTP header:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJNYXJpbyBSb3NzaSIsImVtYWlsIjoibWFyaW8ucm9zc2lAZG9tYWluLmNvbSIsImJpcnRoZGF0ZSI6IjE5ODMtMDktMjMiLCJqdGkiOiJmZjQ0YmVjOC03ZDBkLTQ3ZTEtOWJjZC03MTY4NmQ5Nzk3NzkiLCJleHAiOjE1MTIzMjIxNjgsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6NjM5MzkvIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo2MzkzOS8ifQ.9qyvnhDna3gEiGcd_ngsXZisciNOy55RjBP4ENSGfYI
```

This time we will get the list of books.

## Getting claims

When introduced JWTs, we said that a token may contain some data called *claims*. These are usually information about the user that can be useful when authorizing the access to a resource. Claims could be, for example, user's e-mail, gender, role, city or any other information useful to discriminate users while accessing to resources. We can add claims in a JWT so that they will be available while checking authorization to access a resource. Let's explore in practice how to manage claims in our .NET Core 2 application.

Suppose that our list contains books not suitable for everyone. For example, it contains a book subject to age restrictions. We should include in the JWT returned after the authentication, an information about the user's age. Let's see how to change the code to create a token:

```c#
private string BuildToken(UserModel user)
{

	var claims = new[] {
		new Claim(JwtRegisteredClaimNames.Sub, user.Name),
		new Claim(JwtRegisteredClaimNames.Email, user.Email),
		new Claim(JwtRegisteredClaimNames.Birthdate, user.Birthdate.ToString("yyyy-MM-dd")),
		new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
	   };

	var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
	var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

	var token = new JwtSecurityToken(_config["Jwt:Issuer"],
	  _config["Jwt:Issuer"],
	  claims,
	  expires: DateTime.Now.AddMinutes(30),
	  signingCredentials: creds);

	return new JwtSecurityTokenHandler().WriteToken(token);
}
```

The main differences with respect to the previous version concern the definition of the `claims` variable. It is an array of `Claims` instances, each created from a key and a value. The keys are values of a structure (`JwtRegisteredClaimNames`) that provides names for public [standardized claims](https://tools.ietf.org/html/rfc7519#section-4). We created claims for the user's name, email, birthday and for a unique identifier associated to the JWT.

This `claims` array is then passed to the `JwtSecurityToken` constructor so that it will be included in the JWT sent to the client.

Now, let's take a look at how to change the API code in order to take into account the user's age when returning the list of books:

```c#
[Route("api/[controller]")]
public class BooksController : Controller
{
	[HttpGet, Authorize]
	public IEnumerable<Book> Get()
	{
		var currentUser = HttpContext.User;
		int userAge = 0;
		var resultBookList = new Book[] {
			new Book { Author = "Ray Bradbury", Title = "Fahrenheit 451", AgeRestriction = false },
			new Book { Author = "Gabriel García Márquez", Title = "One Hundred years of Solitude", AgeRestriction = false },
			new Book { Author = "George Orwell", Title = "1984", AgeRestriction = false },
			new Book { Author = "Anais Nin", Title = "Delta of Venus", AgeRestriction = true }
		};

		if (currentUser.HasClaim(c => c.Type == ClaimTypes.DateOfBirth))
		{
			DateTime birthDate = DateTime.Parse(currentUser.Claims.FirstOrDefault(c => c.Type == ClaimTypes.DateOfBirth).Value);
			userAge = DateTime.Today.Year - birthDate.Year;
		}

		if (userAge < 18)
		{
			resultBookList = resultBookList.Where(b => !b.AgeRestriction).ToArray();
		}

		return resultBookList;
	}

	public class Book
	{
		public string Author { get; set; }
		public string Title { get; set; }
		public bool AgeRestriction { get; set; }
	}
}
```

We added the `AgeRestriction` property to the Book class. It is a boolean value that indicates if a book is subject to age restrictions or not.

When a request is received, we check if a claim `DateOfBirth` is associated with the current user. In the affirmative case, we calculate the user's age. Then, if the user is under 18, the list will contain only the books without age restrictions, else the whole list will be returned.

You can test this new scenario by running the tests `GetBooksWithoutAgeRestrictions` and `GetBooksWithAgeRestrictions` included in the [project's source code](https://github.com/andychiare/netcore2-jwt).



## Aside: ????

Auth0’s goal is to provide the [best identity management solution for customers](https://auth0.com/user-management). As a company that primes for security, we have invested a lot of time and effort to develop the best and easiest solution to help companies to properly handle their users' sensitive data. We allow developers to easily extend the security of their applications with features like [Passwordless](https://auth0.com/passwordless), [Multifactor Authentication](https://auth0.com/multifactor-authentication), and [Breached Passwords Detection](https://auth0.com/breached-passwords).

To learn how you can secure your ASP.NET Core application with Auth0, check the [quickstart documentation over here](https://auth0.com/docs/quickstart/webapp/aspnet-core). After that you can learn how to turn on the security features mentioned above by checking the following resources:

* [Multifactor Made Easy](https://auth0.com/multifactor-authentication)
* [Log in without Passwords. Easy and Secure.](https://auth0.com/passwordless)
* [Protect your users and services from password leaks](https://auth0.com/breached-passwords)

> [Auth0 offers a generous **free tier**](https://auth0.com/pricing) to get started with modern authentication.

## Summary

In the article, we had an overview of JSON Web Token technology and introduced how to use it in .NET Core 2. While developing a simple Web API application, we saw how to configure support for JWT authentication and how to create tokens on authentication. We also described how to insert claims into a JWT and how to use them while authorizing the access to a resource.

In conclusion, we experienced how easy is to manage JSON Web Tokens in .NET Core 2.

