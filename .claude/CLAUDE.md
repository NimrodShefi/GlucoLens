# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About Project
GlucoLens is a web app for helping people with Type 1 Diabetes manage their glucose levels. It is done by uploading their glucose data from MiniMed and using AI to provide insights into their data. This is to help them understand their glucose patterns and make better decisions about their health. This is because the data can be difficult to understand sometimes, and knowing what it means can be quite difficult, even when visualised with graphs in PDFs.

This app is not intended to replace medical advice from a doctor or other healthcare professional. It is intended to be a tool for people with Type 1 Diabetes to help them understand their glucose data better. There are no medical advices made by this app.

## Application flow
1. User uploads a MiniMed CSV export.
2. The file is parsed into C# DTOs and the raw file is discarded immediately (never written to disk).
3. Statistics (averages, time-in-range, variability) are computed in plain C#.
4. AI interprets the statistics and DTOs to find glucose trends.
5. Results are shown on the dashboard.

**No persistence.** There is no database. Data lives only for the user's session (the Blazor circuit); closing the tab or refreshing means re-uploading. Do not add storage, caching to disk, or any persistence layer without being asked.

Privacy and no-medical-advice rules are in `.claude/rules/privacy-and-safety.md` and apply to all work. 

## Project architecture

Single web project, organised by capability with folders. Do not split into multiple projects unless asked.

```
GlucoLens/
  Components/            # UI only: upload page, dashboard. Calls Application, contains no logic.
  Models/                # DTOs (GlucoseReading, Bolus, etc.) and AnalysisResult
  DataSources/
    IGlucoseDataSource.cs
    Csv/CsvGlucoseDataSource.cs   # the only code that knows about CSV
  Analysis/
    IGlucoseAnalyser.cs
    Statistics/                   # plain C# calculations (averages, time-in-range)
    Ai/AiGlucoseAnalyser.cs       # prompts and provider client
  Application/
    GlucoseSession.cs             # scoped: holds current data and results
    AnalysisWorkflow.cs           # load -> compute stats -> AI -> store result
```

**Dependency rule:**
- `Models` depends on nothing.
- `DataSources` and `Analysis` depend only on `Models`, never on each other.
- `Application` is the only place that knows about both.
- `Components` talk to `Application` only, never directly to `DataSources` or `Analysis`.

**Swappable data source.** The CSV upload is a stopgap: MiniMed may eventually provide direct database access, so the source must be replaceable without touching analysis or UI code.
- `IGlucoseDataSource` is source-agnostic and returns the domain DTOs.
- CSV headers, format, and parsing quirks stay inside `DataSources/Csv/`. Map CSV columns to DTOs in that one place.
- DTOs model the glucose domain (readings, boluses, basal, carbs, etc.), not the CSV layout.
- A future database source is a new `IGlucoseDataSource` implementation plus one DI registration change.

**Statistics first, AI second.** Compute deterministic numbers (averages, time-in-range, variability) in `Analysis/Statistics/`. The AI interprets and explains patterns; it does not calculate. The dashboard uses the same statistics, and sending aggregates to the AI supports the privacy rules.

**Keep `AnalysisWorkflow` thin.** It only sequences calls (source, statistics, AI, session). Real logic belongs in the capability folder it relates to.

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

## Conventions

- Use `.razor` + `.razor.cs` code-behind when component logic exceeds ~20 lines; otherwise `@code`.
- Style with component-scoped `.razor.css`; no inline styles.
- Session state lives in a `Scoped` service (`GlucoseSession`, one per circuit). Never use a `Singleton` or static field for user data.
- Every capability is used through its interface, registered in `Program.cs`.
- Async end to end; never `.Result` or `.Wait()`.
- Store glucose internally as [mg/dL]; convert to mmol/L only for display. [Confirm]
- Time handling: [decide UTC vs device-local; CSV exports are usually local time without an offset].

## Gotchas

- Interactive Server prerenders by default, so `OnInitializedAsync` can run twice. Keep it idempotent.
- `InputFile` reads are limited to 512 KB by default. Set `maxAllowedSize` explicitly for CSV uploads.
- Scoped state is lost if the circuit drops. The dashboard must handle the "no data loaded" case gracefully.
- SignalR has a default message size limit (32 KB). Don't pass large datasets through JS interop in a single call.

## Project status

Scaffold only; no domain code yet, and no test project. Update this section as features land.
