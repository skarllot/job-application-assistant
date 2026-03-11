# Technology Stack

## Architecture

Vertical slice architecture — each feature is self-contained with its own endpoint, handler, and domain logic. No mediator library; dependencies are consumed through interfaces, registered via DI, enabling loose coupling and straightforward unit testing with NSubstitute.

## Core Technologies

- **Language**: C# with nullable reference types enabled
- **Framework**: Blazor (.NET 10)
- **Runtime**: .NET 10
- **CI/CD**: GitHub Actions

## Key Libraries

- **CSharpier**: Code formatting (maximum line length: 120 characters)
- **xUnit**: Unit and integration testing
- **NSubstitute**: Test mocking/substitution
- **Meziantou.Analyzer**: Additional code quality rules
- **.NET Recommended Analyzers**: Default analyzer set with warnings treated as errors

## Development Standards

### Type Safety

- Nullable reference types enabled project-wide (`<Nullable>enable</Nullable>`)
- Treat all warnings as errors (`<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`)

### Code Quality

- CSharpier for consistent formatting; enforced in CI
- .editorconfig for editor-level style rules
- Maximum line length: 120 characters
- No explanatory code comments — code should be self-documenting
- Use meaningful names, small methods, and clear structure instead of comments

### Testing

- xUnit as the test framework
- NSubstitute for mocking dependencies
- Tests live alongside features in the vertical slice or in a parallel test project

### Static Analysis

- .NET recommended analyzers enabled
- Meziantou.Analyzer for additional quality rules
- All analyzer warnings treated as errors

## Development Environment

### Required Tools

- .NET 10 SDK
- CSharpier (dotnet tool)

### Common Commands

```bash
# Restore: dotnet restore --locked-mode
# Build: dotnet build
# Test: dotnet test
# Format: dotnet csharpier .
# Format check: dotnet csharpier --check .
```

## Key Technical Decisions

- **No mediator pattern**: Dependencies are consumed via interfaces and DI — no mediator indirection
- **Interface-driven design**: All dependencies abstracted behind interfaces for testability and loose coupling
- **Vertical slices over layered architecture**: Each feature owns its full stack (UI, logic, data) for better cohesion and independent evolution
- **Lock file for package restore**: Use `--locked-mode` to ensure deterministic builds
- **Warnings as errors**: Enforce code quality at compile time, no warnings allowed in CI

---
_Document standards and patterns, not every dependency_
