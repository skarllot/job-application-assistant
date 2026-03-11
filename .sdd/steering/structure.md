# Project Structure

## Organization Philosophy

Vertical slice architecture: features are organized by capability, not by technical layer. Each slice contains everything needed to deliver its feature (UI components, handlers, domain logic, data access).

## Directory Patterns

### Feature Slices

**Location**: `src/<ProjectName>/Features/<FeatureName>/`
**Purpose**: Self-contained feature implementations
**Example**: `src/App/Features/ResumeAnalysis/` contains the endpoint, handler, models, and validation for resume analysis

### Shared Domain

**Location**: `src/<ProjectName>/Domain/`
**Purpose**: Cross-cutting domain types shared across features (e.g., value objects, enums)
**Example**: `src/App/Domain/Resume.cs` — domain entity used by multiple features

### Tests

**Location**: `tests/<ProjectName>.Tests/`
**Purpose**: Unit and integration tests mirroring the source structure
**Example**: `tests/App.Tests/Features/ResumeAnalysis/` mirrors the feature slice

## Naming Conventions

- **Files**: PascalCase matching the primary type (`ResumeAnalysisHandler.cs`)
- **Namespaces**: Match directory structure
- **Types**: PascalCase (`ResumeAnalysisHandler`, `CoverLetterRequest`)
- **Methods**: PascalCase (`AnalyzeResume`, `GenerateCoverLetter`)
- **Local variables / parameters**: camelCase (`jobDescription`, `resumeContent`)
- **Private fields**: camelCase with underscore prefix (`_resumeParser`)
- **Interfaces**: `I` prefix (`IResumeParser`)

## Import Organization

```csharp
// System namespaces first, then third-party, then project namespaces
// Managed by .editorconfig and IDE settings — no manual ordering needed
using System.Text.Json;
using Microsoft.AspNetCore.Components;
using JobApplicationAssistant.Domain;
```

## Code Organization Principles

- Each vertical slice is independent; avoid cross-slice dependencies
- Shared code goes in Domain or a shared infrastructure folder only when genuinely reused
- No mediator — slices depend on interfaces registered via DI, not concrete implementations
- Keep slices small; split when a feature grows beyond a single responsibility
- Prefer composition over inheritance

---
_Document patterns, not file trees. New files following patterns shouldn't require updates_
