---
layout: post
title:  "ProblemDetails in ASP.NET Core"
date:   2022-05-04
categories: asp.net core
---

ProblemDetails is a type that provides a standard way of representing errors in HTTP APIs. It was introduced as part of the ASP.NET Core framework and is based on the Internet Engineering Task Force (IETF) draft specification for the "application/problem+json" media type.

The idea behind ProblemDetails is to provide a standard, consistent way of representing errors in HTTP APIs. This can help to ensure that error responses are consistent and understandable, and can make it easier for clients to process and interpret error responses. By using ProblemDetails, developers can ensure that their error responses provide meaningful information to clients, rather than just a simple error code or message.

ProblemDetails is a type that can be used in C# to represent error responses in HTTP APIs. It provides a number of properties that can be used to provide additional information about the error, including the type of error, a title, a detailed description, and the HTTP status code. This information can be used by clients to understand the nature of the error and determine the appropriate action to take.

```csharp
using System;
using System.Net;
using Microsoft.AspNetCore.Mvc;

namespace MyApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class OrderController : ControllerBase
    {
        [HttpGet("{id}")]
        public IActionResult GetOrderInfo(int id)
        {
            // If the order ID is 0, return an error
            if (id == 0)
            {
                // Create a ProblemDetails object to represent the error
                ProblemDetails errorDetails = new ProblemDetails
                {
                    Type = "https://example.com/problems/order-not-found",
                    Title = "Order Not Found",
                    Detail = "The order with the specified ID could not be found.",
                    Status = (int)HttpStatusCode.NotFound
                };

                // Return a NotFound result with the error details
                return NotFound(errorDetails);
            }

            // Return the order information for the specified ID
            return Ok(id);
        }
    }
}
```

As you can see in the example, translating the exceptions into ProblemDetails inside the Controller manually would would be very repetitive and time consuming to write the translation code.

Looking for a standard solution to translating Exceptions into ProblemDetails I came across Khellang's ProblemDetails Middleware.

## Khellang's ProblemDetails Middleware

Handling application exceptions and translating them into ProblemDetails format can be done in ASP.NET Core by using Khellang's ProblemDetails Middleware.

Here's an example of how you can handle exceptions and return error responses in ProblemDetails format:

1. Install the Khellang.AspNetCore.ProblemDetails NuGet package:

    ```bash
    dotnet add package Hellang.Middleware.ProblemDetails
    ```

2. In your Startup.cs file, configure the ProblemDetails Middleware:

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddProblemDetails(ConfigureProblemDetails)
            .AddControllers()
                // Adds MVC conventions to work better with the ProblemDetails middleware.
                .AddProblemDetailsConventions()
                .AddJsonOptions(x => x.JsonSerializerOptions.IgnoreNullValues = true);
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseProblemDetails();

        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }

    ```

3. Configure how the middleware:

    ```csharp
    private void ConfigureProblemDetails(ProblemDetailsOptions options)
    {
        // Only include exception details in a development environment. There's really no need
        // to set this as it's the default behavior. It's just included here for completeness :)
        options.IncludeExceptionDetails = (ctx, ex) => Environment.IsDevelopment();

        // Custom mapping function for FluentValidation's ValidationException.
        options.MapFluentValidationException();

        // You can configure the middleware to re-throw certain types of exceptions, all exceptions or based on a predicate.
        // This is useful if you have upstream middleware that needs to do additional handling of exceptions.
        options.Rethrow<NotSupportedException>();

        // This will map NotImplementedException to the 501 Not Implemented status code.
        options.MapToStatusCode<NotImplementedException>(StatusCodes.Status501NotImplemented);

        // This will map HttpRequestException to the 503 Service Unavailable status code.
        options.MapToStatusCode<HttpRequestException>(StatusCodes.Status503ServiceUnavailable);

        // Because exceptions are handled polymorphically, this will act as a "catch all" mapping, which is why it's added last.
        // If an exception other than NotImplementedException and HttpRequestException is thrown, this will handle it.
        options.MapToStatusCode<Exception>(StatusCodes.Status500InternalServerError);
    }
    ```

Now, note that the three methods below are extension methods on three different interfaces and do different things:

- IMvcBuilder.AddProblemDetailsConventions
    Adds some MVC-specific services and filters to make MVC work better with the middleware.
    This must be called on an IMvcBuilder instance. You get one by calling AddMvc, AddControllers, AddControllersWithViews or AddRazorPages. The order here shouldn't matter either.

- IApplicationBuilder.UseProblemDetails
    Adds the actual middleware to the application pipeline.
    This should normally happen as early as possible, to make sure no middleware can throw without it being handled and transformed into a problem details response.

- IServiceCollection.AddProblemDetails
    Adds the required services for the middleware to work and must always be called.
    The order shouldn't matter.

## Conclusion

In summary, ProblemDetails provides a standard way of representing errors in HTTP APIs, making it easier for clients to process and understand error responses. This can help to ensure that error responses are consistent, meaningful, and actionable, and can help to improve the overall user experience for clients consuming the API.

Links:

[InvalidModelStateResponseFactory - issue #2](https://github.com/khellang/Middleware/issues/2)
[ApiBehaviorOptions docs](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.apibehavioroptions?view=aspnetcore-5.0)

SuppressMapClientErrors issues:
https://github.com/khellang/Middleware/issues/75
https://github.com/khellang/Middleware/issues/120

AddProblemDetailsConventions:
https://github.com/khellang/Middleware/issues/121
