---
name: vertical-slice
description: Use this skill when the user asks to create, generate, build, or scaffold a new API endpoint, feature, or vertical slice in an ASP.NET Core project using Minimal APIs.
---

# Vertical Slice Generator

## Architecture & File Structure Rules
- **Access Modifiers:** All generated types (the main slice class, Request/Response DTOs, Validator, Endpoint, and Handler) MUST be strictly `internal`. Do NOT use `public` for types. The ONLY exception is methods implementing interfaces (like `public void MapEndpoint`), which must be `public`.
- **Single File:** The entire feature (Request, Response, Validator, Endpoint, and optionally Handler) MUST be enclosed within a single `internal static class FeatureName`.
- **MediatR is FORBIDDEN:** Do NOT use `IMediator` or `IRequestHandler`.
- **No Repositories:** Do NOT use the Repository pattern. Inject the specific application DbContext (e.g., `AppDbContext`), NEVER the base `Microsoft.EntityFrameworkCore.DbContext`. Obtain the exact DbContext class name from the project's `AGENTS.md` context or the user's prompt.
- **Dependency Injection & Auto-Registration:** You MUST explicitly use the `[FromServices]` attribute for ALL application services injected into the Minimal API `Handle` method parameters (e.g., `[FromServices] IValidator<Request> validator`, `[FromServices] AppDbContext db`, `[FromServices] Handler handler`). **CRITICAL EXCEPTION:** Do NOT use `[FromServices]` on framework-provided parameters such as `CancellationToken`, `HttpContext`, or `ClaimsPrincipal` — they are resolved automatically. Do NOT rely on implicit DI resolution for your custom services. Do NOT manually register types in `Program.cs`. Ensure the `Handler` (if used) explicitly implements the custom `IScopedType` interface to guarantee automatic DI registration. Do NOT implement `IScopedType` on the `Validator`, as it assumes `services.AddValidatorsFromAssembly()` is configured in the project.
- **Async & Cancellation:** All endpoint and Handler methods MUST be asynchronous. The Endpoint's `Handle` method MUST return `Task<IResult>`. The `Handler` (if used) MUST NOT return `IResult` directly; it should return a domain outcome (e.g., a DTO or a Result pattern) as defined by the project's `AGENTS.md`. You MUST inject a `CancellationToken` into the entry point and pass it down to ALL asynchronous operations (especially EF Core database calls).
- **Fail-first (Early Return) Pattern:** You MUST use guard clauses and handle all error cases, null checks, and failures first across ALL methods (Endpoints, Handlers, etc.). Return the corresponding error or failure result immediately (e.g., if (entity is null), if (result.IsFailure)). Do not use elseblocks for the main execution path. The successful outcome (e.g.,Results.Ok, Result.Success) MUST be the very last line of the method.

## Context Fallbacks & Dependencies
- **DbContext Fallback:** If `AGENTS.md` is missing or you cannot determine the exact `DbContext` name, scan the `Infrastructure` or `Data` directories for a class inheriting from `Microsoft.EntityFrameworkCore.DbContext`. If still not found, halt and ask the user.
- **Mandatory Usings:** You MUST include `using FluentValidation;` and `using FluentValidation.Results;` (the latter is strictly required for the `.ToDictionary()` extension method).
- **Custom Interfaces:** For the custom `IScopedType` and `IEndpoint` interfaces, infer their namespaces by scanning the project (usually in a Shared, Core, or Infrastructure building block). If you cannot find them, explicitly ask the user before generating the code.
- **Result Pattern & Error Handling:** The specific implementation of the Result pattern (e.g., `ErrorOr`, `FluentResults`, or custom wrappers) and how to map them to HTTP status codes MUST be obtained from the project's `AGENTS.md`. If missing, assume a standard generic wrapper. Do NOT blindly return HTTP 200 OK if a Handler returns a Result object containing an error. **CRITICAL: Only use the Result pattern when interacting with a `Handler`. For Simple Logic (Skeleton 1) where you query the DbContext directly, do NOT wrap the outcome in a Result pattern; just return the DTO directly (e.g., `Results.Ok(dto)`).**

## Logic Placement Rule (Endpoint vs. Handler)
You must decide where to put the business logic based on its complexity:
- **Simple Logic (Keep in Endpoint):** If the feature consists of basic CRUD operations, 1-2 straightforward database queries, and simple mapping. If the flow is just "read request -> query DB -> return response", write the logic DIRECTLY inside the `private static async Task<IResult> Handle(...)` method of the nested `Endpoint` class.
- **Complex Logic (Extract to Handler):** Extract the logic into a nested `Handler` class ONLY IF the feature requires complex business rules, deeply nested conditions, explicit database transactions, or needs to inject multiple external dependencies (e.g., Email services, external APIs) alongside the `DbContext`. Inject this `Handler` into the `Endpoint.Handle` method.

## Component Specifications
- **Endpoint & Routing:** Create a nested `internal sealed class Endpoint : IEndpoint`. Map the route inside `public void MapEndpoint(IEndpointRouteBuilder app)`. You MUST use standard RESTful HTTP methods (GET, POST, PUT, DELETE). Route paths MUST be in `kebab-case` and logically structured (e.g., `/api/user-profiles`). NEVER use PascalCase or camelCase in the URL path. Append `.WithTags("FeatureGroup")` to the endpoint mapping to organize OpenAPI documentation. Infer the tag name from the domain context.
- **Responses & DTOs:** NEVER return raw Domain Entities directly to the client. Always **manually** map complex types to DTOs (e.g., a nested `Response` record) before returning them from the endpoint or handler. Do NOT use AutoMapper, Mapster, or any other mapping libraries. **CRITICAL EXCEPTION:** If the endpoint returns only a single value (e.g., a single string token, an int ID, List<T> items, or a bool), do NOT wrap it in a Response DTO record. Return the type directly (e.g., Results.Ok(token)).
- **Request Binding:** You MUST strictly follow Minimal API binding rules based on the HTTP method. Use `[FromBody]` for POST, PUT, and PATCH requests. You MUST use `[AsParameters]` for GET and DELETE requests when using a complex `Request` record, because Minimal APIs cannot bind complex types from the request body for these methods by default.
- **HTTP Status Codes:** You MUST return appropriate HTTP status codes based on the RESTful operation and the execution outcome:
  - Return `Results.Ok(...)` (200) for successful GET requests or when returning modified data.
  - Return `Results.Created(...)` (201) for successful POST requests that create a new resource.
  - Return `Results.NoContent()` (204) for successful DELETE or PUT/PATCH requests that do not return a body.
  - Return `Results.NotFound()` (404) if a requested entity does not exist in the database.
  - Return `Results.Conflict(...)` (409) or `Results.BadRequest(...)` (400) for domain rule violations.
  - In .Produces() methods use StatusCodes static class, **DO NOT USE HARDCODED STATUS CODE NUMBERS.**
- **Validation:** If the endpoint accepts user input, create a nested `internal sealed class Validator : AbstractValidator<Request>` using FluentValidation. You MUST inject the Validator as its interface `IValidator<Request>` (NEVER the concrete `Validator` class) and call `.ValidateAsync(request, ct)`. If validation fails, map the errors using: `return Results.ValidationProblem(validationResult.ToDictionary());`.
- **Namespaces:** Infer the correct namespace from the project structure. Do NOT hardcode namespaces. Include any domain-specific standard usings found in the project.

## Required File Structure (Skeleton)
You MUST structure the file exactly like one of the following templates. Choose the appropriate skeleton based on the Logic Placement Rule.

### Skeleton 1: Simple Logic (No Handler)
Use this when the logic is simple enough to reside directly in the Endpoint. Do NOT generate a Handler class.

```csharp
using FluentValidation;
using FluentValidation.Results;
// Include other necessary usings (EF Core, Domain entities, Custom Interfaces)

namespace Inferred.Namespace.Here;

internal static class FeatureName
{
    internal sealed record Request(string Data);

    // Create a Response record ONLY if returning multiple fields. 
    // If returning a single (e.g., string token, int Id, List<T> Items), delete this record and return the type directly.
    internal sealed record Response(string Result);

    internal sealed class Validator : AbstractValidator<Request>
    {
        public Validator()
        {
            // RuleFor(x => x.Data)...
        }
    }

    internal sealed class Endpoint : IEndpoint
    {
        public void MapEndpoint(IEndpointRouteBuilder app)
        {
            app.MapPost("/api/feature-name", Handle)
                // adjust type if no Response record
                .Produces<Response>()
                .ProducesValidationProblem()
                .ProducesProblem(StatusCodes.Status404NotFound)
                .ProducesProblem(StatusCodes.Status409Conflict)
                .WithTags("FeatureGroup");
        }

        private static async Task<IResult> Handle(
            [FromBody] Request request, // Use [AsParameters] instead if this is a GET/DELETE request
            [FromServices] IValidator<Request> validator,
            [FromServices] AppDbContext db,
            CancellationToken cancellationToken)
        {
            var validationResult = await validator.ValidateAsync(request, cancellationToken);
            if (!validationResult.IsValid)
            {
                return Results.ValidationProblem(validationResult.ToDictionary());
            }

            // Write simple DbContext query and logic here
            // var entity = await db.Entities.FirstOrDefaultAsync(...);

            // FAIL-FIRST check
            // if (entity is null)
            // {
            //     return Results.NotFound();
            // }

            // SUCCESS: Match the HTTP status code to the REST operation.
            // For MapPost (Create):
            // Note: Replace '{newId}' with the actual ID property of the created entity.
            return Results.Created($"/api/feature-name/{newId}", new Response("Processed"));
            
            // For MapGet / MapPut (Read / Update):
            // return Results.Ok(new Response("Processed"));
            
            // For MapDelete (Delete):
            // return Results.NoContent();
        }
    }
}
```

### Skeleton 2: Complex Logic (With Handler)
Use this when the logic is complex and requires a separate Handler.

```csharp
using FluentValidation;
using FluentValidation.Results;
// Include other necessary usings (EF Core, Domain entities, Custom Interfaces)

namespace Inferred.Namespace.Here;

internal static class FeatureName
{
    internal sealed record Request(string Data);

    // Create a Response record ONLY if returning multiple fields. 
    // If returning a single (e.g., string token, int Id, List<T> Items), delete this record and return the type directly.
    internal sealed record Response(string Result);

    internal sealed class Validator : AbstractValidator<Request>
    {
        public Validator()
        {
            // RuleFor(x => x.Data)...
        }
    }

    internal sealed class Endpoint : IEndpoint
    {
        public void MapEndpoint(IEndpointRouteBuilder app)
        {
            app.MapPost("/api/feature-name", Handle)
                // adjust type if no Response record
                .Produces<Response>()
                .ProducesValidationProblem()
                .ProducesProblem(StatusCodes.Status404NotFound)
                .ProducesProblem(StatusCodes.Status409Conflict)
                .WithTags("FeatureGroup");
        }

        private static async Task<IResult> Handle(
            [FromBody] Request request, // Use [AsParameters] instead if this is a GET/DELETE request
            [FromServices] IValidator<Request> validator,
            [FromServices] Handler handler, 
            CancellationToken cancellationToken)
        {
            var validationResult = await validator.ValidateAsync(request, cancellationToken);
            if (!validationResult.IsValid)
            {
                return Results.ValidationProblem(validationResult.ToDictionary());
            }

            var response = await handler.HandleAsync(request, cancellationToken);
            
            // FAIL-FIRST: Handle errors before success
            
            // IF using a Result pattern (dictated by AGENTS.md), handle failures like this:
            // if (response.IsFailure)
            // {
            //     return Results.BadRequest(response.ErrorDetails!.ErrorMessage); 
            // }
            
            // IF returning plain DTOs, handle edge cases (e.g., nulls):
            // if (response is null)
            // {
            //     return Results.NotFound();
            // }

            // SUCCESS: Match the HTTP status code to the REST operation.
            // Use response.Value! if using Result pattern, otherwise use response directly.
            
            // For MapPost (Create):
            // Note: Replace '{newId}' with the actual ID property from the response.
            return Results.Created($"/api/feature-name/{newId}", response);
            
            // For MapGet / MapPut (Read / Update):
            // return Results.Ok(response);
            
            // For MapDelete (Delete):
            // return Results.NoContent();
        }
    }
    
    internal sealed class Handler(AppDbContext db) : IScopedType
    {
        // If the project uses a Result pattern, change the return type (e.g., Task<Result<Response>>)
        public async Task<Response> HandleAsync(Request request, CancellationToken cancellationToken)
        {
            // Complex domain logic and transaction handling here
            
            return new Response("Processed");
        }
    }
}
```

## Workflow & Execution
- **Context Gathering**: Before writing any code, you MUST read the `AGENTS.md` file (if it exists in the project). Explicitly extract the exact `DbContext` name, the `Result` pattern wrapper (if project uses this pattern) (e.g., ErrorOr, FluentResults, custom), custom interfaces, and namespace conventions. Do NOT guess these values.
- **Plan**: Analyze the request and decide if a separate Handler class is needed based on complexity. Output this decision.
- **Generate**: Write the complete feature code in a single file based on the rules above.
- **Save**: Save the file in the appropriate feature directory requested by the user.
- **Verify**: Run **dotnet build** on the project. If there are compiler errors (like missing using directives or type names), read the error, fix the code, and recompile. **Maximum 3 attempts**. If the build still fails after the 3rd attempt, you MUST STOP immediately. Do NOT make any further code changes, do NOT guess the solution, and explicitly output the final compilation error message to the user. **Do NOT auto-commit.**
