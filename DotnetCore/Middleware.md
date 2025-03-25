# Middleware
- Middleware intercepts incoming requests & outgoing response & allow to modify them.
- Middleware allow to add functionality to your application without changing underlying code.
- It can be used in wide area like Authentication, logging, compression, caching etc.
- Middleware sits between Web server & your Application. As when a request come to web server, it passed through series of middleware before it reaches the application, Similarly response from application goes through series of middleware before back to client.
- 
## Default middleware in ASP.net core
- ASP.net core provide set of default middleware which are executed in specific order to handle different concern like routing, authentication, error handling. Below are few default middlewares:
### 1. Exception Handling Middleware (UseExceptionHandler)
- It is responsible for handling unhandled exception of the application during request processing.
- When error occure, it can be configured to show Error page or redirect to error handling endpoint: `app.UseExceptionHandler("/Home/Error");`
### 2. HTTPS Middleware (UseHsts() and UseHttpsRedirection())
- They both enforce use of HTTPS (Hyper Text Transfer Protocol Secure) for our application, but they are used at different stages.
#### 2.1 UseHttpsRedirection() Middleware
- This middleware automatically redirects all HTTP incoming request to HTTPS.
- It doesn't add any header like UseHsts() and just redirect.
- This middleware relies on server-side redirection and cannot prevent initial HTTP requests like HSTS.
#### 2.2 UseHsts() Middleware
- `UseHsts()` enforces **HSTS (HTTP Strict Transport Security)** policy, which tells browser to use **HTTPS** for a specified period.
- This is done by adding **Strict Transport Security** HTTP header with specified duration in `max-age` in respose to browser.
`Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`  
	- `max-age=31536000` → Enforce HTTPS for 1 year.
	- `includeSubDomains` → Applies to subdomains as well.
	- `preload` → Allows browsers to preload the rule (optional).
- It does not automatically redirect HTTP to HTTPS (you need UseHttpsRedirection() for that).
- Only works after the first request (since the browser must first receive the HSTS header).
- Example:
```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseHsts(); // Enforces HTTPS at the browser level
}

app.UseHttpsRedirection(); // Redirects HTTP requests to HTTPS
```
### 3. Static Files Middleware (UseStaticFiles)
- This middleware serve static files like CSS, images, javascript & other resources from "wwwroot" folder by default.
- Example, When you access wwwroot/image.jpg, this middleware will serve the file directly  
  `    app.UseStaticFiles();  // Serves static files from wwwroot `
### 4. Routing Middleware (UseRouting)
- It routes incoming HTTP requests to appropriate endpoints (controllers, Razor pages, etc.) by looking at the URL and determines which controller and action (or page) to invoke.
`    app.UseRouting();`
### 5. Authentication Middleware (UseAuthentication)
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
#### Common Authentication Schemes
##### a) JWT Bearer Authentication
- If you're using JWT tokens (e.g., OAuth2, IdentityServer), you can configure the authentication middleware like this:

##### b) Cookie Authentication: 
- If you're using cookie-based authentication.
```cs
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
    });
```
##### c) API Key or Custom Authentication: 
- You can also create your custom authentication scheme, such as API key authentication.


### 6. Authorization Middleware (UseAuthorization)
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
- Under Authorization, we can configure different Roles, Claims & Policies which can be applied to different controllers or actions using `[Authorize]` attribute.
- Example, below we have added 2 Policies (AdminPolicy & ManagerPolicy) and 2 Roles (Admin & Manager) and 1 Claim (CanEdit) to Authorization service.
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add Authentication service to builder
builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer(options =>
    {
        options.Authority = "https://your-identity-server";
        options.Audience = "your-api";
    });

// Add Authorization service to builder
builder.Services.AddAuthorization(options =>
{
    // Configure a role-based policy
    options.AddPolicy("AdminPolicy", policy => 
    {
        policy.RequireRole("Admin");
    });

    // Configure a custom policy combining roles and claims
    options.AddPolicy("ManagerPolicy", policy =>
    {
        policy.RequireRole("Manager");
        policy.RequireClaim("CanEdit", "true");
    });
});

builder.Services.AddControllers();

var app = builder.Build();

// Enable authentication and authorization middleware
app.UseAuthentication();  // Authenticate the request
app.UseAuthorization();   // Authorize the request

// Map controllers or endpoints
app.MapControllers();

app.Run();
```
- Now `[Authorize]` attribute can be used to apply them on different controllers or actions.
```csharp
[Authorize(Roles = "Admin")]  // Only users with "Admin" role can access this controller
public class AdminController : ControllerBase {....}

[Authorize(Policy = "ManagerPolicy")]  // Only users who meet the "ManagerPolicy" can access this controller
public class ManagerController : ControllerBase {...}

//Using Multiple Policies in [Authorize]
[Authorize(Policy = "AdminOnly")]  // Requires the "AdminOnly" policy
[Authorize(Policy = "ManagerPolicy")]  // Requires the "ManagerPolicy" policy
public class MultiPolicyController : ControllerBase


public class HomeController : ControllerBase
{
    [HttpGet]
    [Authorize(Roles = "Admin")]  // Only Admins can access this action
    public IActionResult GetAdminData()
    {
        return Ok("This is admin data.");
    }

    [HttpGet]
    [Authorize(Policy = "ManagerPolicy")]  // Only users meeting ManagerPolicy can access this action
    public IActionResult GetManagerData()
    {
        return Ok("This is manager data.");
    }
}
```
### 7. Endpoitnt Middleware (UseEndpoints())
- Endpoint middleware executes the endpoint (i.e. controller action or razor page etc) matched to the routes defined by `UseRouting()` middleware. Hence `UseEndpoint()` is used after `UseRouting()` middleware.
- Whereas `UseRouting()` middleware creates routes matching the incoming Http request based on the Url, Http method etc.
```csharp
app.UseRouting();        // Enables endpoint routing
app.UseAuthentication(); // If authentication is needed
app.UseAuthorization();  // Enables authorization policies
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers(); // This maps controller actions to routes
});
```

### Sequence order of all middleware
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage(); // Show detailed error page in development
}
else
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts(); // Secure HTTP headers
}

app.UseHttpsRedirection(); // Redirect to HTTPS
app.UseStaticFiles(); // Serve static files

app.UseRouting(); // Start the routing process

app.UseAuthentication(); // Authenticate requests
app.UseAuthorization();  // Authorize requests

app.MapControllers(); // Map controllers to routes

app.Run();
```
## Types of Middleware
- There are two types of middleware. Terminal & Non-Terminal.

### 1) Terminal Middleware 
- It is the final middleware component in the pipeline as there will be no middleware after it. 
- It can be used to modify outgoing response before it sent to the client. Examples:-
	- **Static file middleware**: used to serve static files such as CSS, javascript, images.	`app.UseStaticFiles();`
	- **File server middleware**: it serve files from specified directory.
	- **MVC middleware**: it is used to handle  request of MVC endpoints.

### 2) Non-Terminal Middleware
- It is not the final middleware component in the pipeline.
- It can be used to modify incoming request & outgoing response. Examples:-
	- **Authentication middleware**: it is used to authenticate user.
	- **Authorization middleware**: to authorize user.
	- **Routing middleware**: to request route to appropriate end point.	`app.UseRouting();`
	
## Middleware Extension method
- We have multiple extension methods for IApplicationBuilder
### 1) Use()
- this extension method add new custom middleware component which will call also call next middleware component in the pipeline.
- It takes two input parameters, one is HttpContext context which can access both incomint request & outgoing response to be processed by middleware, and other is Func<Task> delegate which represent next middleware component to be called. 
- Below custom middleware will just print message on browser & then call next middleware.
### 2) Next()
- this extention method call the next middleware component in the pipeline.
### 3) Run()
- this extention method add a Terminal middleware component in pipeline which will not call any other middleware. Example:-
- Below custom middleware will just print message on browser & return response back to client.
### 4) MapGet()
- it define an endpoint for the incoming request.

### Below we have use of Use(), Run(), MapGet() extension methods:
```csharp
using System.Globalization;

var builder = WebApplication.CreateBuilder(args);
//IApplicationBuilder instance
var app = builder.Build();

app.UseHsts();
app.UseHttpsRedirection();
app.UseResponseCaching();
app.UseResponseCompression();
app.UseStaticFiles();
app.UseHttpLogging();
app.UseRouting();

app.UseAuthentication();
app.UseAuthorization();
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.Use(async (context, next) => 
{
	await context.Response.WriteAsync("This is custom middleware");
	//call next middleware component in pipeline
	await next();
});

app.UseMiddleware<CustomMiddleware>(); 

//Print message if incoming url contain /hello
app.MapGet("/hello", () => "Hello World!");

app.MapControllers();

app.Run(async (context) =>
{
    await context.Response.WriteAsync($"This is a Terminal middleware");
});
```

## Custom Middleware
There are multiple ways to create custom middleware:
### 1. Using IMiddleware interface
- We can create our own custom middleware component by creating a class which implements the IMiddleware interface.
- It has a single method InvokeAsync which takes two parameter: HttpContext & Func<Task> delegate which represent next middleware component to be invoked.
- Below we created a Custom middleware which adds custom header "X-Custom-Header" in response to client. 
```csharp
public class CustomMiddleware : IMiddleware
{
	public async Task InvokeAsync(HttpContext context, Func<Task> next)
	{
		//Add custom header in response.
		context.Response.Header.Add("X-Custom-Header", "Hello world");
		//call next middleware component in pipeline
		await next();
	}
}
```

- And later included new middleware component to the pipeline using `UseMiddleware()` method of IApplicationBuilder.
```csharp
//add CustomMiddleware component to pipeline
app.UseMiddleware<CustomMiddleware>(); 
```

### 2. Using Inline Middleware
- To create inline middleware we use extension methods Use(), Next() & Run() of IApplicationBuilder which will create custom middleware in Program.cs only without creating separate class. This is shown above.

### 3. Using RequestDelegate
- 

## Middleware Best Practices
- Use middleware only when needed, as though it is a powerful tool but using it too much in pipeline will slow down the application.
- Order middleware components carefully to achieve desired sequence.
- Use terminal middleware components when appropriate as no further middleware component will be executed afterwards.



