---
name: vertical-slice
description: Scaffolds a new Vertical Slice feature in a single static file using Minimal APIs and direct DbContext.
---

# Vertical Slice Generator

## Architecture & File Structure Rules
- **Access Modifiers:** All generated types (the main slice class, Request/Response DTOs, Validator, Endpoint, and Handler) MUST be strictly `internal`. Do NOT use `public` for types. The ONLY exception is methods implementing interfaces (like `public void MapEndpoint`), which must be `public`.
- **Single File:** The entire feature (Request, Response, Validator, Endpoint, and optionally Handler) MUST be enclosed within a single `internal static class FeatureName`.
- **MediatR is FORBIDDEN:** Do NOT use `IMediator` or `IRequestHandler`.
- **No Repositories & DbContext:** Do NOT use the Repository pattern. Inject the specific application DbContext (e.g., `AppDbContext`), NEVER the base `Microsoft.EntityFrameworkCore.DbContext`. Obtain the exact DbContext class name from the project's `AGENTS.md` context or the user's prompt.
- **Dependency Injection & Auto-Registration:** Inject dependencies via `[FromServices]` in the Minimal API endpoint method, or via primary constructor if using a separate Handler class. Do NOT manually register the Handler or Endpoint in the DI container. The `IScopedType` and `IEndpoint` interfaces handle registration automatically. NEVER modify `Program.cs`.
- **Async & Cancellation:** All endpoint and Handler methods MUST be asynchronous. The Endpoint's `Handle` method MUST return `Task<IResult>`. The `Handler` (if used) MUST NOT return `IResult` directly; it should return a domain outcome (e.g., a DTO or a Result pattern) as defined by the project's `AGENTS.md`. You MUST inject a `CancellationToken` into the entry point and pass it down to ALL asynchronous operations (especially EF Core database calls).

## Context Fallbacks & Dependencies
- **DbContext Fallback:** If `AGENTS.md` is missing or you cannot determine the exact `DbContext` name, scan the `Infrastructure` or `Data` directories for a class inheriting from `Microsoft.EntityFrameworkCore.DbContext`. If still not found, halt and ask the user.
- **Mandatory Usings:** You MUST include `using FluentValidation;` and `using FluentValidation.Results;` (the latter is strictly required for the `.ToDictionary()` extension method).
- **Custom Interfaces:** For the custom `IScopedType` and `IEndpoint` interfaces, infer their namespaces by scanning the project (usually in a Shared, Core, or Infrastructure building block). If you cannot find them, explicitly ask the user before generating the code.
- **Result Pattern:** If the Handler returns a Result pattern and its definition is not found in `AGENTS.md`, assume a standard generic wrapper or ask the user for the specific implementation used in the project.

## Logic Placement Rule (Endpoint vs. Handler)
You must decide where to put the business logic based on its complexity:
- **Simple Logic (Keep in Endpoint):** If the feature consists of basic CRUD operations, 1-2 straightforward database queries, and simple mapping. If the flow is just "read request -> query DB -> return response", write the logic DIRECTLY inside the `private static async Task<IResult> Handle(...)` method of the nested `Endpoint` class.
- **Complex Logic (Extract to Handler):** Extract the logic into a nested `Handler` class ONLY IF the feature requires complex business rules, deeply nested conditions, explicit database transactions, or needs to inject multiple external dependencies (e.g., Email services, external APIs) alongside the `DbContext`. Inject this `Handler` into the `Endpoint.Handle` method.

## Component Specifications
- **Endpoint & Routing:** Create a nested `internal sealed class Endpoint : IEndpoint`. Map the route inside `public void MapEndpoint(IEndpointRouteBuilder app)`. You MUST use standard RESTful HTTP methods (GET, POST, PUT, DELETE). Route paths MUST be in `kebab-case` and logically structured (e.g., `/api/user-profiles`). NEVER use PascalCase or camelCase in the URL path. Append `.WithTags("FeatureGroup")` to the endpoint mapping to organize OpenAPI documentation. Infer the tag name from the domain context.
- **Responses & DTOs:** NEVER return raw Domain Entities directly to the client. Always **manually** map complex types to DTOs (e.g., a nested `Response` record) before returning them from the endpoint or handler. Do NOT use AutoMapper, Mapster, or any other mapping libraries.
- **Validation:** If the endpoint accepts user input (via `[FromBody]`, `[FromQuery]`, route parameters, or `[AsParameters]`), create a nested `internal sealed class Validator : AbstractValidator<Request>` using FluentValidation. You MUST inject the Validator and call `.ValidateAsync(request, ct)`. If validation fails, map the errors using the built-in extension method: `return Results.ValidationProblem(validationResult.ToDictionary());`.
- **Namespaces:** Infer the correct namespace from the project structure. Do NOT hardcode namespaces. Include any domain-specific standard usings found in the project.

## Required File Structure (Skeleton)
You MUST structure the single file exactly like this template:

```csharp
using FluentValidation;
using FluentValidation.Results;
// Include other necessary usings (EF Core, Domain entities, Custom Interfaces)

namespace Inferred.Namespace.Here;

internal static class FeatureName
{
    internal sealed record Request(string Data);
    
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
                .WithTags("FeatureGroup");
        }

        private static async Task<IResult> Handle(
            Request request,
            Validator validator,
            // Inject Handler here IF using complex logic, OR inject AppDbContext directly for simple logic
            Handler handler, 
            CancellationToken ct)
        {
            var validationResult = await validator.ValidateAsync(request, ct);
            if (!validationResult.IsValid)
            {
                return Results.ValidationProblem(validationResult.ToDictionary());
            }

            // Call handler.HandleAsync OR write simple DbContext query here
            var response = await handler.HandleAsync(request, ct);

            return Results.Ok(response);
        }
    }
    
    // Include Handler ONLY if logic is complex (see Logic Placement Rule)
    internal sealed class Handler(AppDbContext db) : IScopedType
    {
        public async Task<Response> HandleAsync(Request request, CancellationToken ct)
        {
            // Complex domain logic here
            return new Response("Processed");
        }
    }
}
```
