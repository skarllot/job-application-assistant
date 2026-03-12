# Implementation Plan

- [x] 1. Build and SDK Configuration
  - Create SDK version pinning with .NET 10, disallowing pre-release SDKs and allowing only patch-level roll-forward
  - Create shared build properties enabling nullable reference types, warnings as errors, recommended analysis level, and lock file generation for all projects
  - _Requirements: 1.4, 1.5, 1.6, 2.4_

- [x] 2. (P) Dependency Management
  - Set up centralized package version management with a root configuration file enabling CPM
  - Create source-scoped package versions including Meziantou.Analyzer as a global package reference applied to all source projects
  - Create test-scoped package versions for xUnit v3, xUnit runner, NSubstitute, and Microsoft.NET.Test.Sdk
  - Each child configuration imports the root to inherit the CPM setting
  - _Requirements: 2.3, 3.1, 3.2, 3.3_

- [x] 3. (P) Code Quality Tooling
- [x] 3.1 (P) Set up CSharpier formatting tool
  - Create a dotnet local tool manifest registering CSharpier at the specified version
  - Create a CSharpier configuration file setting maximum line width to 120 characters
  - _Requirements: 2.1, 2.5_

- [x] 3.2 (P) Create editor configuration for code style rules
  - Define naming conventions: PascalCase for types, methods, and properties; camelCase for locals and parameters; underscore-prefixed camelCase for private fields; I-prefix for interfaces
  - Set indentation to 4 spaces, charset to UTF-8, and configure using directive ordering and placement
  - Scope formatting rules to complement CSharpier (naming and semantic rules only, not layout)
  - _Requirements: 2.2_

- [x] 4. Solution and Blazor Web Application
- [x] 4.1 Create solution and web project
  - Create a .NET solution file at the repository root
  - Create a Blazor web application project under src/ targeting .NET 10 with the Web SDK
  - Add the web project to the solution
  - Create an empty Features directory as a placeholder for future vertical slices
  - _Requirements: 1.1, 1.2_

- [x] 4.2 Implement Blazor application shell
  - Create the root component with HTML document structure and Blazor script references
  - Create a router component for client-side navigation
  - Create a layout with a header displaying "Job Application Assistant" and a main content area
  - Create a landing page mapped to the root URL
  - Create shared Razor imports for common directives
  - Configure the application entry point with interactive server-side rendering services and middleware
  - Create launch settings with a configured HTTP port
  - _Requirements: 5.1, 5.2, 5.3, 5.4_

- [x] 5. Test Project and Infrastructure
  - Create a test project under tests/ targeting .NET 10, referencing the web project
  - Add the test project to the solution
  - Include package references for xUnit, NSubstitute, and test SDK (versions managed centrally)
  - Create a passing smoke test that validates the test infrastructure is functional
  - _Requirements: 1.3, 6.1, 6.2, 6.3, 6.4_

- [x] 6. CI/CD Pipeline
  - Create a GitHub Actions workflow triggered on push and pull request events, running on Ubuntu
  - Configure steps: checkout, set up .NET SDK from the global.json file, restore dotnet tools, check formatting with CSharpier, cache NuGet packages keyed on lock files, restore dependencies in locked mode, build in Release configuration with no redundant restore, and run tests with no redundant build
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

- [x] 7. Generate Lock Files and Validate
  - Restore all packages to generate lock files for each project
  - Run CSharpier to format all generated code with zero violations
  - Build the full solution and confirm zero warnings
  - Run all tests and confirm they pass
  - _Requirements: 2.5, 3.1, 3.2, 6.4_
