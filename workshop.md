# Beginners Guide to OpenID Connect in Angular/.NET Application

> This workshop is about implementing OIDC in an Angular/C# app. Read the links if you need more details.

- Goal 1: Create a simple Angular app, using ASP.NET Core Web Api in the Backend
- Goal 2: Implement the BFF architecure
- Goal 3: Add OpenID Connect (OIDC) to this app

Lots of input from Damien Bowden: <https://damienbod.com/2023/09/25/secure-angular-application-using-openiddict-and-asp-net-core-with-bff/>

## Definitions

- STS: Secure Token Service (openIddict or IdentityServer)
- OIDC: OpenID Connect <https://openid.net/developers/how-connect-works/>
- BFF: "Backend For Frontend" Architecture
- BE: Backend (ASP.NET Core WebApi)
- FE: Frontend (Angular)

## New to `.NET` and`C#`

- Download and Install .NET
- Download and install Visual Studio (as for now, for .NET I prefer the full version of "Visual Studio", not "Visual Stuidio Code")
- Details <https://learn.microsoft.com/en-us/dotnet/core/install/windows>
- I use C# for .NET <https://learn.microsoft.com/en-us/dotnet/csharp/>

## New to Angular

- Install node.js LTS (installs node and npm): <https://nodejs.org/en>
- Install Angular CLI: `npm install -g @angular/cli`
- I use "Visual Studio Code" for Angular development: <https://code.visualstudio.com/>
- Details: <https://angular.dev/tutorials/first-app>

## About BFF

BFF = "Backend For Frontend" Architecture

> BFF: Security is handled in the Backend

Main Advantages of BFF:

- Login, Logout are initiated from BE (usually requested from FE)
- Tokens *) are stored in BE (not unsecure FE)
- Security is implemented in BE (there we have the access token)

*) Tokens contain security information. They are issued from STS.

Other Advantages of BFF:

- Authentication handled in BE
- BE serves FE
- BE + FE on the same port: example: `<backend-dns:backend-port>/index.html`
- no Security Tokens (sent from STS) in FE (more secure)
- NO Authentication in FE: no library in FE needed
- ASP.NET Core delivers all needed functionality (no additional libraries needed)
- FE is a secure partner for BE (using same-site cookie)
  - Same-Site-Cookies: protects against CSRF = Cross Site Request Forgery attacks
- BFF is the recommended/most current state of the industry
- Details: <https://learn.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends>

Disadvantages of BFF:

- BE + FE need to run on the same port: this needs a reverse proxy (YARP in our case) in development process only, because Angular DEV webserver shares the web app on a different port. (in fact it is only needed for Angular debugging. Otherwise simple copy - like shown here - works fine)

## About STS

>STS: A service, which handles the login for your application

- similar things: IdP (Identity Provider), IAM (Identity and Access Management), ...
- examples: <https://openid.net/certification/#OPENID-OP-P>
- related: Single-Sign-On (SSO)
- old technologies: Custom login page, Windows Authentication, ... <https://www.pac4j.org/blog/a-brief-history-of-the-security-protocols.html>
- old protocols: SAML, CAS, OpenID, OAuth <https://www.pac4j.org/blog/a-brief-history-of-the-security-protocols.html>
- current protocol: OpenID Connect (OIDC) uses OAuth 2.0 + OpenID 2.0

### Why to we need a local STS?

In theory, we could use the company's STS, but this might have some issues

- localhost not supported in our company
- needs to be configured for developers
- a local STS can be configured as needed (to find and fix CORS or other issues)
- helpful for better understanding of OIDC/OAuth

### Identity Server

- I this example, we use Identity Server as STS
- it implements OAuth 2.0 + OpenID 2.0
- free for development (but do not use any libraries in the clients, they are not free)
- simple to create
- simple to use
- works out of the box: OIDC api, login page, logout page
- simple to config: config files for users and apps
- plenty of docs and blogs available
- if you need something free to use in production: use OpenIddict

### OIDC = OpenID Connect

OIDC: OpenID Connect <https://openid.net/developers/how-connect-works/>

### OIDC Flows

>BFF uses "Code Flow"

- Flow: a way to get tokens --> we use "Code Flow" for scenario App+STS
- Diagrams of all the OpenID Connect Flows: <https://darutk.medium.com/diagrams-of-all-the-openid-connect-flows-6968e3990660>
- Hint: every few years, the recommended flows change - you need to keep up with the current security standards

## What we are going to implement now

- Create .NET core WebApi (BE)
- Create Angular project (FE)
- Implement BFF: Integrate FE in BE
- Create STS (Identity Server)
- Implement Authentication, Login, Logout

## Folder structure

In the end, we'll have this folder structure. sln file, is above and it contains both .NET projects (IdentityServerForDev, MyBackend) and the Angular project (MyFrontend)

```cmd
\MyOidcBffDemo
    MyOidcBffDemo.sln
    \IdentityServerForDev
    \MyBackend
    \MyFrontend
```

## Solution and add Backend (ASP.NET Core WebApi)

What the following script does:

1. create a new solution
2. create a new app
3. add app to solution

```cs
dotnet new sln -o MyOidcBffDemo
cd MyOidcBffDemo
dotnet new webapi --name MyBackend
dotnet sln add ./MyBackend/MyBackend.csproj
```

Test api, there are 2 ways:

> Note! Make sure you use 7138 for https: open \Properties\launchSettings.json, otherwise

- in the https section
- remove htttp URL
- change https url to port to 7138
- optional: change launchBrowser to true (to launch a browser on start)
- optional: remove http section

If you want, you can remove all the obsolete profiles in this file and set https+port like this:

```json
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "profiles": {
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "https://localhost:7138",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```
Test the API:
- Open solution in Visual studio, press F5 to run the Backend:
  - see json data in https://localhost:7138/weatherforecast

## Create new Frontend (Angular) App

Make sure cmd is in `MyOidcBffDemo`:

```cmd
npm install -g @angular/cli
ng new MyFrontend --skip-git --style css --ssr false
```

Test the Angular app:

```cmd
cd MyFrontend
ng s -o
cd ..
```

## Config BFF

### add index.html

Add UseStaticFiles to `program.cs`:

```cs
...
app.UseHttpsRedirection(); // existing
app.UseDefaultFiles(); // to load index.html on /
app.UseStaticFiles(); // added: reads files from wwwroot (call https://localhost:7138/index.html)
...
```

- create folder \wwwroot in \MyBackend
- add a index.html (contains 'hello world') file to \wwwroot
- run app and verify hello world: <https://localhost:7138/>
- You will see a page with a big "Hello, MyFrontend"

### Now we need to integrate FE into BE

### Option a): Compile (build) Angular to wwwroot

This is simple too simple and just for understanding. Better use Option b).

The big disadvantage of this option: no Angular debug is avilable, because we need to build the app after each change to run it in BE.

The following change ov `angular.json` will change the output folder of Angular build to /MyBackend/wwwroot.

In FE, edit `angular.json`:

```json
// old:
//"outputPath": "dist/my-frontend",

// new:
"outputPath": {
    "base": "../MyBackend/wwwroot",
    "browser": ""
},
```

In cmd in the folder `\MyFrontend`, run `npm run build` to build/compile your FE.
Run Backend and open <<https://localhost:7138/> to see the Angular Demo page, served by the backend.
You will see "Hello, MyFrontend" again, but now served from BE.

> Congratulations, you implemented your first (and most simple) BFF.

### Option b): use Angular Development to run in Backend

- <https://github.com/damienbod/bff-openiddict-aspnetcore-angular>
- maybe also <https://damienbod.com/2023/09/18/secure-angular-application-using-auth0-and-asp-net-core-with-bff/>

Install yarp:

```
dotnet add package Yarp.ReverseProxy
```

Add middleware:

```
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));
var app = builder.Build();
if (app.Environment.IsDevelopment())
{
    app.MapReverseProxy();
}
app.Run();
```

Add Config to appsettings.Development.json

```json
"ReverseProxy": {
    "Routes": {
        "route1": {
            "ClusterId": "cluster1",
            "Match": {
                "Path": "{**catch-all}"
            }
        }
    },
    "Clusters": {
        "cluster1": {
            "HttpClient": {
                "SslProtocols": [
                    "Tls12"
                ],
                "DangerousAcceptAnyServerCertificate": true
            },
            "Destinations": {
                "cluster1/destination1": {
                    "Address": "http://localhost:4200/"
                }
            }
        }
    }
}
```

Run both: FE and BE. BE (Port 7138) appears in browser but will show FE (running Port 4200).
NOTE: Check console to see the data in from BE.

```
npm run start
F5 in Visual Studio
```

> Advantage: no ssl cert is needed for FE

### Frontend: get data from backend

This `\src\app\app.ts` was generated:

```ts
import { Component, signal } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  templateUrl: './app.html',
  styleUrl: './app.css'
})
export class App {
  protected readonly title = signal('MyFrontend');
}
```

Replace it with this new AppComponent (add http client module, call /weatherforecast api)

```ts
import { HttpClient, HttpClientModule } from '@angular/common/http'; // added
import { Component, signal, OnInit } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, HttpClientModule], // added: HttpClientModule
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class App implements OnInit {
  // HttpClient needs import of HttpClientModule (add to imports)
  constructor(private readonly http: HttpClient) {}
  ngOnInit(): void {
    this.http.get('/weatherForecast').subscribe((res) => {
      console.warn("Got data from BE", res); // Check browser debugger console to see the data
    });
  }
  protected readonly title = signal('MyFrontend');
}
```

For a more enhanced version (html table) check this: <https://stackoverflow.com/questions/69084631/fetch-data-on-asp-net-core-with-angular-template-stuck-on-loading-vs>

Now Build FE app again using `npm run build`, run the app using `https://localhost:7138/index.html` and check console in your browser debugger tool:
you see the weatherforecast data in the console.

## OIDC - OpenID Connect

- Implement a safe way to secure your FE and BE: modern solution: OIDC (OpenID Connectd)
- OIDC is about Authentication (allow user to login), not Authorization (allow user to do something in the app)
- OIDC needs an STS (Secure Token Service). In development, we will use IdentityServer from Duende.
- Login/Logout is handled in STS
- Each application must be configured in STS (Users, and for each client: CliendID, password (optional), allowed callback URL)
- Well documented STS for development: OpenIddict, IdentityServer

>Why do I use a local STS? A local STS is handy for development (and the company STS does not support local debugging - localhost is not supported/blocked)

## OpenIddict (fyi - not part of this workshop)

OpenIddict example: <https://damienbod.com/2023/09/25/secure-angular-application-using-openiddict-and-asp-net-core-with-bff/>

## Create new STS (IdentityServer, Duende)

>- Note 1: Read about the licence: IdentityServer is free - for DEV only! (need a free library for PROD? check OpenIddict)
>- Note 2: Beware: a lot of blogs/documents in the Internet use libraries from IdentityServer/Duende: they are not free! This document uses free libraries only.

Instructions from <https://github.com/DuendeSoftware/IdentityServer.Templates>

Install or updates templates:
`dotnet new install Duende.IdentityServer.Templates`

- Create IdentityServer in folder 'MyOidcBffDemo\IdentityServerForDev'
- We use this template: isinmen = Duende IdentityServer with In-Memory Stores and Test Users
- Add the new project to the solution:

```cmd
md IdentityServerForDev
cd IdentityServerForDev
dotnet new isinmem
cd ..
dotnet sln add ./IdentityServerForDev/IdentityServerForDev.csproj
```

- check the code (class TestUsers, class Config)
- run it, click any `here` on the 'Duende IdentityServer' page to login using bob/bob or alice/alice

From now on, run both projects: Backend, Identity Server:

- right-click solution, configure startup project, Multiple startup project: select both with Action 'Start'

## Secure/protect API

Simplest version: add `[Authorize]` to Endpoint:

Minimal API: before:

```cs
app.MapGet("/weatherforecast", () =>
```

Minimal Api: after:

```cs
app.MapGet("/weatherforecast", [Authorize] () =>
```

Regular API: old:

```cs
[HttpGet(Name = "GetWeatherForecast")]
public IEnumerable<WeatherForecast> Get()
{
...
```

Regular API: new:

```cs
[Authorize]
[HttpGet(Name = "GetWeatherForecast")]
public IEnumerable<WeatherForecast> Get()
{
...
```

Add required services to your BE (/MyBackend) to enable Authorization:

```cs
var builder = WebApplication.CreateBuilder(args); // already there
builder.Services.AddAuthentication(); // add this: required for [Authorize]
builder.Services.AddAuthorization(); // add this:  required by AddAuthentication
...
```

Add Authorization middleware (UseAuthorization):

```cs
...
app.UseHttpsRedirection(); // already there
app.UseStaticFiles(); // already there
app.UseAuthorization();  // add this: required for [Authorize]
...
```

Now run app again, and you should not see the weatherforecast data, because we proteted it. But you'll see this error in the console:

`System.InvalidOperationException: No authenticationScheme was specified, and there was no DefaultChallengeScheme found. The default schemes can be set using either AddAuthentication(string defaultScheme) or AddAuthentication(Action<AuthenticationOptions> configureOptions).`

## Config BE to use OIDC STS

> We use our local IdentityServer now

Add OpenIdConnect client package to your BE (/MyBackend):

`dotnet add package Microsoft.AspNetCore.Authentication.OpenIdConnect`

Add the following OIDC configuration (please have a closer look to Authority, ClientId, ClientSecret):

```cs
// add this to your BE, after builder.Services.AddAuthorization() and before builder.Build();:
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options =>
{
    options.Authority = "https://localhost:5001"; // STS (URL of your IdentityServer)
    options.ClientId = "interactive";  // found/configured in sts (IdentityServer: Config.cs)
    options.ClientSecret = "49C1A7E1-0C79-4A89-A3D6-A37998FB86B0"; // found/configured in sts (IdentityServer: Config.cs)
    options.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.SaveTokens = true;
    options.GetClaimsFromUserInfoEndpoint = true;
    options.TokenValidationParameters = new TokenValidationParameters
    {
        NameClaimType = "name"
    };
});
```

Change Client url's (of Client where ClientId = "interactive") in STS (Config.cs):

old/existing urls from template:

```cs
RedirectUris = { "https://localhost:44300/signin-oidc" },
FrontChannelLogoutUri = "https://localhost:44300/signout-oidc",
PostLogoutRedirectUris = { "https://localhost:44300/signout-callback-oidc" },
```

new: use our client/backend URL:

```cs
RedirectUris = { "https://localhost:7138/signin-oidc" },
FrontChannelLogoutUri = "https://localhost:7138/signout-oidc",
PostLogoutRedirectUris = { "https://localhost:7138/signout-callback-oidc" },
```

Enable conventional (not minimal) controllers: add AddControllers, MapControllers:

```cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers(); // added for AccountController
...
app.MapControllers(); // added for AccountController
app.Run();
```

Add AccountController.cs in the folder \Controllers of your BE

a) full api

```cs
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

[AllowAnonymous]
[Route("api/account")]
public class AccountController : ControllerBase
{
    [AllowAnonymous]
    [HttpGet("Login")]
    public ActionResult Login()
    {
        AuthenticationProperties properties = new() { RedirectUri = "/", };
        ChallengeResult ret = Challenge(properties); // if not loggeed in: redirect to STS
        return ret;
    }

    [Authorize(AuthenticationSchemes = CookieAuthenticationDefaults.AuthenticationScheme)]
    [HttpPost("Logout")]
    public IActionResult Logout()
    {
        return SignOut(
            new AuthenticationProperties { RedirectUri = "/" },
            CookieAuthenticationDefaults.AuthenticationScheme
        );
    }
}
```

b) Same for minimal API

```
// Login endpoint (anonymous)
app.MapGet("/api/account/login", [AllowAnonymous] async (HttpContext context) =>
{
    var props = new AuthenticationProperties { RedirectUri = "/" };
    // await context.ChallengeAsync(props); // if not logged in: redirect to STS -- doppelt 
    return Results.Challenge(props);
});

// Logout endpoint (authorized)
app.MapPost("/api/account/logout", [Authorize(AuthenticationSchemes = CookieAuthenticationDefaults.AuthenticationScheme)] async (HttpContext context) =>
{
    await context.SignOutAsync(
        CookieAuthenticationDefaults.AuthenticationScheme,
        new AuthenticationProperties { RedirectUri = "/" }
    );
    return Results.SignOut(
        new AuthenticationProperties { RedirectUri = "/" },
        new[] { CookieAuthenticationDefaults.AuthenticationScheme }
    );
});
```

Identity Server: configure CORS to allow access

a) Programm.cs

old:

```cs
    ...
    var app = builder ...
    ...
```

new:

```cs
    ...
    builder.Services.AddCors(options =>
    {
        options.AddPolicy(name: "_myAllowSpecificOrigins",
            policy =>
            {
                _ = policy.WithOrigins("https://localhost:7138").AllowAnyHeader().AllowAnyMethod(); // OIDC client URL
            });
    });

    var app = builder ...
    ...
```

b) HostingExtensions.cs

Add `UseCors()` to ConfigurePipeline

```cs
app.UseCors("_myAllowSpecificOrigins");
```

BE: Configure CORS in BE to allow Access from Identity Server

add this somewhere in the build part (before builder.Bild()):

```cs
string MyAllowSpecificOrigins = "_myAllowSpecificOrigins";

builder.Services.AddCors(options =>
{
    options.AddPolicy(name: MyAllowSpecificOrigins,
        policy =>
        {
            _ = policy.WithOrigins("https://localhost:5001").AllowAnyHeader().AllowAnyMethod(); // STS URL
        });
});
```

add `UseCors` :

```cs
...
app.UseCors(MyAllowSpecificOrigins);
app.UseHttpsRedirection(); // existing line
...
```

## Config FE to call Login and Logout

Add Login (get) and Logout (post) to Frontend. Add this at the end of `app.html` after `<router-outlet />`:

```html
 <h2>Login/Logout:</h2>
 <br />
 <a class="btn btn-link" href="api/Account/Login">Log in</a>
 <br />
 <form
   method="post"
   action="api/Account/Logout">
   <button class="btn btn-link" type="submit">Sign out</button>
 </form>
```

build FE: in `\MyFrontend` run `npm run build` and start Backend (Press F5 in Visual Studio).

## Test the Login/Logout

Start BFF, open web page: <https://localhost:7138/> and click Login.
You get redirected to STS: Login using bob/bob or alice/alice (see file TestUsers.cs in IdentityServer).

Check Chrome Developer Tools:

- network: redirect call to signin-oidc (<https://localhost:7138/signin-oidc>) returns Status 302. Check the scopes: profile and sopenid
- Backend call to protected api (<https://localhost:7138/weatherForecast>) returns Status 200. Check the 5 data records which have been returned

Press Logout. The refresh tha page: there is no weatherforecast output in the console.

Press Login again: no login requried/no login page apears: STS sill knows you.

fyi: There are some JavaScript errors - it looks like they are comming from Identity Server. Ignore it - unless they are CORS related.

## TODO minor things/Next steps

- fix logout
- cors restrict (now all is open)
- add more security (like PKCE: see bowden)
- login in swagger
- json parse error when not logged in
