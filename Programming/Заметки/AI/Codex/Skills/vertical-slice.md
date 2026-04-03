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
- **Dependency Injection:** Inject dependencies via `[FromServices]` in the Minimal API endpoint method, or via primary constructor if using a separate Handler class.
- **Async & Cancellation:** All endpoint methods (`Handle`) and Handler methods MUST be asynchronous (`async Task<...>`, `async Task<IResult>`). You MUST inject a `CancellationToken` into the entry point and pass it down to ALL asynchronous operations (especially EF Core database calls).

## Logic Placement Rule (Endpoint vs. Handler)
You must decide where to put the business logic based on its complexity:
- **Simple Logic (Keep in Endpoint):** If the feature consists of basic CRUD operations, 1-2 straightforward database queries, and simple mapping. If the flow is just "read request -> query DB -> return response", write the logic DIRECTLY inside the `private static async Task<IResult> Handle(...)` method of the nested `Endpoint` class.
- **Complex Logic (Extract to Handler):** Extract the logic into a nested `Handler` class ONLY IF the feature requires complex business rules, deeply nested conditions, explicit database transactions, or needs to inject multiple external dependencies (e.g., Email services, external APIs) alongside the `DbContext`. Inject this `Handler` into the `Endpoint.Handle` method.

## Component Specifications
- **Endpoint & Routing:** Create a nested `internal sealed class Endpoint : IEndpoint`. Map the route inside `public void MapEndpoint(IEndpointRouteBuilder app)`. You MUST use standard RESTful HTTP methods (GET, POST, PUT, DELETE). Route paths MUST be in `kebab-case` and logically structured (e.g., `/api/user-profiles`). NEVER use PascalCase or camelCase in the URL path.
- **Responses & DTOs:** NEVER return raw Domain Entities directly to the client. Always **manually** map complex types to DTOs (e.g., a nested `Response` record) before returning them from the endpoint or handler. Do NOT use AutoMapper, Mapster, or any other mapping libraries.
- **Validation:** If the endpoint accepts a payload (`[FromBody]`), create a nested `internal sealed class Validator : AbstractValidator<Request>` using FluentValidation. You MUST inject the Validator and call `.ValidateAsync(request, ct)`. If validation fails, map the errors using the built-in extension method: `return Results.ValidationProblem(validationResult.ToDictionary());`.
- **Namespaces:** Infer the correct namespace from the project structure. Do NOT hardcode namespaces. Include any domain-specific standard usings found in the project.

## Workflow & Execution
1. **Plan:** Analyze the request and decide if a separate `Handler` class is needed based on complexity. Output this decision.
2. **Generate:** Write the complete feature code in a single file based on the rules above.
3. **Save:** Save the file in the appropriate feature directory requested by the user.
4. **Verify:** Run `dotnet build` on the project. If there are compiler errors (like missing using directives or type names), read the error, fix the code, and recompile. Maximum 3 attempts. Do NOT auto-commit.
