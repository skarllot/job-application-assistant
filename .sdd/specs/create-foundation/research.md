# Research & Design Decisions

## Summary
- **Feature**: `create-foundation`
- **Discovery Scope**: New Feature (greenfield scaffolding)
- **Key Findings**:
  - .NET 10 is an LTS release; Blazor Web App uses unified rendering model with `AddRazorComponents().AddInteractiveServerComponents()` for interactive server-side rendering
  - xUnit v3 (3.2.2) is the current major version, recommended over v2 for new projects; uses `xunit.v3` meta-package
  - Central Package Management (CPM) with `Directory.Packages.props` is the standard approach for multi-project solutions

## Research Log

### .NET 10 Blazor Web App Template Structure
- **Context**: Need to understand the default project structure for a Blazor Web App with Interactive Server rendering in .NET 10
- **Sources Consulted**: [ASP.NET Core Blazor project structure](https://learn.microsoft.com/en-us/aspnet/core/blazor/project-structure?view=aspnetcore-10.0), [Blazor render modes](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/render-modes?view=aspnetcore-10.0)
- **Findings**:
  - Blazor Web App template (`blazor`) is the single project template for all render modes
  - For Interactive Server only, a single project is sufficient (no `.Client` project needed)
  - Key structure: `Components/` folder containing `App.razor`, `Routes.razor`, `NotFound.razor`, `Layout/` subfolder (MainLayout, NavMenu, ReconnectModal), `Pages/` subfolder
  - Program.cs configuration: `builder.Services.AddRazorComponents().AddInteractiveServerComponents()` and `app.MapRazorComponents<App>().AddInteractiveServerRenderMode()`
  - .NET 10 adds `NotFound.razor` and `ReconnectModal` component by default
  - `_Imports.razor` for shared directives, `wwwroot/` for static assets
- **Implications**: The standard template structure aligns well with the vertical slice architecture — features will live under `Features/` alongside the default `Components/` structure

### xUnit v3 Setup
- **Context**: Requirements specify xUnit as test framework; need to determine the correct package setup
- **Sources Consulted**: [xUnit v3 NuGet packages](https://xunit.net/docs/nuget-packages-v3), [NuGet Gallery](https://www.nuget.org/packages/xunit.v3/)
- **Findings**:
  - xUnit v3 latest: 3.2.2 (January 14, 2026)
  - Main package: `xunit.v3` — includes framework, analyzers, and assertions
  - For `dotnet test` support: `xunit.runner.visualstudio` (VSTest adapter)
  - Supports .NET 8.0+ (fully compatible with .NET 10)
  - v3 is recommended for new projects over v2
- **Implications**: Use `xunit.v3` and `xunit.runner.visualstudio` packages in the test project

### NSubstitute Version
- **Context**: Requirements specify NSubstitute for mocking
- **Sources Consulted**: [NuGet Gallery](https://www.nuget.org/packages/nsubstitute/)
- **Findings**: Latest version is 5.3.0
- **Implications**: Include NSubstitute 5.3.0 in Directory.Packages.props

### CSharpier Configuration
- **Context**: Requirements specify CSharpier as formatting tool with 120 character line length
- **Sources Consulted**: [CSharpier installation](https://csharpier.com/docs/Installation), [NuGet Gallery](https://www.nuget.org/packages/CSharpier)
- **Findings**:
  - Latest version: 1.2.6 (February 6, 2026)
  - Installed as dotnet local tool via `.config/dotnet-tools.json` manifest
  - Configuration via `.csharpierrc.yaml` file
  - CI check: `dotnet csharpier --check .`
- **Implications**: Install as local tool, configure 120 char line width in `.csharpierrc.yaml`

### Meziantou.Analyzer
- **Context**: Requirements specify Meziantou.Analyzer for additional code quality rules
- **Sources Consulted**: [NuGet Gallery](https://www.nuget.org/packages/Meziantou.Analyzer/), [GitHub](https://github.com/meziantou/Meziantou.Analyzer)
- **Findings**: Latest version is 3.0.15 (February 23, 2026); Roslyn analyzer for enforcing C# best practices
- **Implications**: Reference as `PrivateAssets="all"` analyzer in source projects via Directory.Build.props or Directory.Packages.props

### Central Package Management
- **Context**: Requirements mandate centralized package version management
- **Sources Consulted**: [Microsoft Learn CPM](https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management)
- **Findings**:
  - `Directory.Packages.props` at repo root with `<ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>`
  - `<PackageVersion>` items define versions centrally
  - Project files use `<PackageReference>` without `Version` attribute
  - Lock files generated per project with `<RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>`
- **Implications**: All package versions centralized; projects reference packages without version attributes

### GitHub Actions for .NET 10
- **Context**: Requirements specify CI pipeline with GitHub Actions
- **Sources Consulted**: [GitHub Actions .NET overview](https://learn.microsoft.com/en-us/dotnet/devops/github-actions-overview), [actions/setup-dotnet](https://github.com/actions/setup-dotnet)
- **Findings**:
  - `actions/setup-dotnet` supports `dotnet-version: 10.0.x`
  - Standard workflow: checkout → setup-dotnet → restore → build → test → format check
  - NuGet caching available via `actions/cache`
- **Implications**: Straightforward workflow; use `--locked-mode` for restore, `--no-restore` for build/test steps, `dotnet csharpier --check .` for format validation

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| Vertical Slice | Features self-contained with own UI, logic, data | High cohesion, independent evolution, minimal cross-cutting | Potential duplication of shared patterns | Aligns with steering tech.md and structure.md |
| Layered | Traditional separation by technical concern | Well understood, clear separation | High coupling between layers, harder to evolve independently | Does not align with steering |

## Design Decisions

### Decision: xUnit v3 over v2
- **Context**: Need to choose xUnit version for test infrastructure
- **Alternatives Considered**:
  1. xUnit v2 (2.9.3) — stable, widely used
  2. xUnit v3 (3.2.2) — latest major version, recommended for new projects
- **Selected Approach**: xUnit v3 via `xunit.v3` meta-package
- **Rationale**: New project with no legacy constraints; v3 is the recommended starting point and supports .NET 10 natively
- **Trade-offs**: v3 has slightly different API surface from v2, but as a greenfield project this is irrelevant
- **Follow-up**: Ensure `xunit.runner.visualstudio` is included for `dotnet test` compatibility

### Decision: Single Blazor Project (no .Client)
- **Context**: Choose between single project or split server/client project for Blazor Web App
- **Alternatives Considered**:
  1. Single project with Interactive Server only
  2. Split project with .Client for WebAssembly support
- **Selected Approach**: Single project with Interactive Server render mode only
- **Rationale**: Requirements specify interactive server-side rendering; no WebAssembly requirement. Simpler project structure for the foundation phase
- **Trade-offs**: Cannot use WebAssembly rendering without restructuring later, but this aligns with current requirements
- **Follow-up**: If WebAssembly is needed later, a .Client project can be added

### Decision: Split Directory.Packages.props by Project Group
- **Context**: Source and test projects have entirely disjoint package dependencies; a single file mixes unrelated concerns
- **Alternatives Considered**:
  1. Single `Directory.Packages.props` at root listing all packages
  2. Root file for shared config, child files in `src/` and `tests/` for group-specific packages
- **Selected Approach**: Hierarchical split — root sets `ManagePackageVersionsCentrally`, `src/` and `tests/` each have their own `Directory.Packages.props` importing the root and defining group-specific package versions
- **Rationale**: Keeps source and test dependencies clearly scoped. Adding a test package does not touch the source configuration and vice versa. CPM evaluates the closest file to each project, and child files import the root via `MSBuild::GetPathOfFileAbove`
- **Trade-offs**: Three files instead of one; slightly more initial setup. Justified because the package sets have zero overlap
- **Follow-up**: If shared packages emerge later (e.g., a logging library used by both src and tests), they can be added to the root file

### Decision: SDK Version Pinning via global.json
- **Context**: Need to ensure consistent .NET SDK version across local development and CI, and prevent use of pre-release SDKs
- **Alternatives Considered**:
  1. Specify `dotnet-version` directly in GitHub Actions workflow
  2. Use `global.json` at repo root and reference it from CI via `global-json-file`
- **Selected Approach**: `global.json` with `allowPrerelease: false` and `rollForward: latestPatch`; CI uses `global-json-file: global.json` in `actions/setup-dotnet`
- **Rationale**: Single source of truth for SDK version — local `dotnet` CLI and CI both read from the same file, eliminating version drift. `allowPrerelease: false` prevents accidental use of preview SDKs
- **Trade-offs**: Requires updating `global.json` when upgrading SDK, but this is an explicit, reviewable change
- **Follow-up**: None

### Decision: Analyzer Configuration via Directory.Build.props
- **Context**: Need to enable analyzers consistently across all source projects
- **Alternatives Considered**:
  1. Per-project PackageReference for analyzers
  2. Centralized in Directory.Build.props with conditional inclusion
- **Selected Approach**: Centralized in Directory.Build.props with conditions to target source projects
- **Rationale**: Ensures consistency, reduces duplication, follows CPM pattern
- **Trade-offs**: All source projects get the same analyzers; fine for this project's size
- **Follow-up**: Use condition to exclude test projects from Meziantou.Analyzer if needed

## Risks & Mitigations
- .NET 10 may still be in preview depending on timeline — Mitigation: use latest stable preview SDK; .NET 10 GA expected November 2025 (already released as LTS)
- xUnit v3 API differences from v2 examples — Mitigation: follow official v3 documentation
- CSharpier formatting may conflict with .editorconfig rules — Mitigation: CSharpier is opinionated and overrides style; .editorconfig covers naming and other rules CSharpier does not touch

## References
- [ASP.NET Core Blazor project structure](https://learn.microsoft.com/en-us/aspnet/core/blazor/project-structure?view=aspnetcore-10.0)
- [ASP.NET Core Blazor render modes](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/render-modes?view=aspnetcore-10.0)
- [xUnit v3 NuGet packages](https://xunit.net/docs/nuget-packages-v3)
- [CSharpier installation](https://csharpier.com/docs/Installation)
- [Central Package Management](https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management)
- [GitHub Actions for .NET](https://learn.microsoft.com/en-us/dotnet/devops/github-actions-overview)
- [actions/setup-dotnet](https://github.com/actions/setup-dotnet)
- [Meziantou.Analyzer](https://github.com/meziantou/Meziantou.Analyzer)
- [NSubstitute](https://nsubstitute.github.io/)
