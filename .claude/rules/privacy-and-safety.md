# Privacy and safety

These rules apply to all work in this repository. Glucose data is sensitive health data, and this app must never give medical advice.

## No medical advice

- AI output describes patterns and observations only (e.g. "glucose tends to rise after breakfast on weekdays").
- Never produce dosing, insulin, carb-ratio, medication, or treatment recommendations, and never tell the user what they should change about their therapy.
- Prompts sent to the AI must instruct it to follow this rule, and AI output must not be shown without the disclaimer below.
- Always show a visible disclaimer wherever AI insights appear: this is not medical advice, and the user should consult a healthcare professional about their care.
- Phrase insights as observations the user can discuss with their care team, not as instructions.

## Health data privacy

- Never log raw glucose readings, insulin or carb values, timestamps tied to readings, or any patient identifiers (names, IDs, device serial numbers). Log counts, durations, and error types only.
- Never commit real MiniMed exports. Test fixtures must be synthetic and live in a clearly named `TestData/` folder. If a file might contain real data, do not add it to the repo.
- Never put real data in prompts, comments, issues, commit messages, or examples.
- Uploaded CSV files are processed in memory and discarded immediately. Never write them to disk, temp folders included.
- Parsed data lives only in session-scoped (per-circuit) state and is cleared when the session ends. Never store it in a singleton, static field, cache, or any shared location where another user could read it.
- Do not add persistence, telemetry, analytics, or third-party tracking that could capture glucose data.

## Sending data to the AI provider

- Send the minimum needed. Prefer aggregates and summaries (e.g. hourly averages, time-in-range) over raw readings where it still answers the question.
- Strip identifiers (names, device serial numbers, account details) before sending anything.
- Never expose API keys in source control, client-side code, or logs. Use `dotnet user-secrets` in development and environment variables in production.

## Errors and validation

- Validate uploads: accept CSV only, enforce a maximum file size, and show a clear, friendly error for malformed data.
- Error messages shown to users or written to logs must not echo file contents or row data.
