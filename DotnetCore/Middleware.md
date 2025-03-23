# Default middleware in ASP.net core
- ASP.net core provide set of default middleware which are executed in specific order to handle different concern like routing, authentication, error handling. Below are few default middlewares:
## 1. Exception Handling Middleware (UseExceptionHandler)
- It is responsible for handling unhandled exception of the application during request processing.
- When error occure, it can be configured to show Error page or redirect to error handling endpoint: `app.UseExceptionHandler("/Home/Error");`
## 2. Static Files Middleware (UseStaticFiles)
- This middleware serve static files like CSS, images, javascript & other resources from "wwwroot" folder by default.
- Example, When you access wwwroot/image.jpg, this middleware will serve the file directly 
```cs    app.UseStaticFiles();  // Serves static files from wwwroot ```
## 3. Routing Middleware (UseRouting)
- It routes incoming HTTP requests to appropriate endpoints (controllers, Razor pages, etc.) by looking at the URL and determines which controller and action (or page) to invoke.
`    app.UseRouting();`
## 4. Authentication Middleware (UseAuthentication)
- It is responsible for enabling authentication for all incoming Http request for application to know who the user is.
- First this middleware will look for the Authentication scheme configured for authenticating the requests like JWT token, Cookie-based, IdentityServer, API key or custom.
- Secondly, it will vaildate the credentials extracted above against the scheme configured.
- After authentication the request, this middleware create a `ClaimPrincipal` object which will represent the current authenticated user. The object contain identity & claims of the user (like roles, permission etc).
- Then the middleware will store the `ClaimPrincipal` in the `HttpContext.User` property so that same can be accessed by the application.
- This middleware is essential for **authorization** check (like `[Authorize]` attribute or policy based authorisation) since it ensures that `HttpContext.User` object is populated.
- The default Authentication scheme is **"No Authentication"**, if we don't explicitly configure any authentication scheme. If we don't configure any Authentication scheme, .net core won't authenticate the request and `HttpContext.User` will be `null` unless some other middleware adds the user manually.
- Example:
```cs
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddAuthentication("Bearer")  // Add authentication services, such as JWT bearer tokens
    .AddJwtBearer(options => {
        options.Authority = "https://example.com";
        options.Audience = "api";
    });

var app = builder.Build();

// Enable Authentication middleware
app.UseAuthentication();

// Ensure authorization middleware is after authentication
app.UseAuthorization();

// Map controllers or endpoints
app.MapControllers();

app.Run();
```
### Common Authentication Schemes
#### a) JWT Bearer Authentication
- If you're using JWT tokens (e.g., OAuth2, IdentityServer), you can configure the authentication middleware like this:

#### b) Cookie Authentication: 
- If you're using cookie-based authentication.
```cs
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
    });
```
#### c) API Key or Custom Authentication: 
- You can also create your custom authentication scheme, such as API key authentication.


## 5. Authorization Middleware (UseAuthorization)
- Authorization middleware ensures authenticated user has needed permissions to access the application using below steps:
- First, it check user is authenticated by ensuring that `HttpContext.User` contain the valid `ClaimPrincipal` object.
- Then, it check the user has required Permissions (like Admin or User), Claims (like "CanEditProfile", "MakePayment") or Policies.
- Also it return **HTTP 403 Forbidden**, if user authenticated but lack necessary permissions.
- And it return **HTTP 401 Unauthorized** status, if user not authenticated.
- `UseAuthorization()` is used after `UseAuthentication()` middleware in the pipeline.
```cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Configure middleware
app.UseAuthentication();  // Authenticate the request
app.UseAuthorization();   // Authorize the request
```


