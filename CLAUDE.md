# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

GlucoLens is a freshly scaffolded Blazor Web App (from the .NET template) with no domain code yet. There is no test project, README, or lint configuration so far.

## Commands

Run from the repo root (the solution file is `GlucoLens.slnx`, the newer XML solution format, which needs a recent .NET SDK):

```sh
dotnet build GlucoLens.slnx
dotnet run --project GlucoLens                         # http profile: http://glucolens.dev.localhost:5291
dotnet run --project GlucoLens --launch-profile https  # https://glucolens.dev.localhost:7023
dotnet watch --project GlucoLens                       # hot reload
```

## Architecture

- **Target:** .NET 10 (`net10.0`), with nullable reference types and implicit usings enabled.
- **Render mode:** Interactive Server is enabled **globally**. `App.razor` puts `@rendermode="InteractiveServer"` on `<Routes>` and `<HeadOutlet>`, so every page runs over a SignalR circuit. Pages don't need their own `@rendermode`. Component code runs on the server, so it can use server-side services directly.
- **Routing:** `Components/Routes.razor` uses `MainLayout` as the default layout. `Pages/NotFound.razor` is the router's `NotFoundPage`, and `Program.cs` also re-executes status-code responses to `/not-found`. `/Error` is the exception handler outside Development.
- **`BlazorDisableThrowNavigationException=true`** (csproj): `NavigationManager.NavigateTo` during static rendering does not throw `NavigationException`.
- **Static assets:** served through `MapStaticAssets()` and referenced in markup through `@Assets["..."]`, which fingerprints the URLs. Component-scoped CSS (`*.razor.css`) is bundled into `GlucoLens.styles.css`.
- **Namespaces:** add new global Razor usings to `Components/_Imports.razor`.
