# Meta-Prompt Optimizer: Self-Refining Prompt Compiler

A client-side web app that turns a rough, one-line intent into a structured, production-style system prompt. It runs entirely in the browser with React 18 and Babel standalone, so there is no build step, no backend, and no page reloads.

**Version:** `v2.4-deterministic`

## Description

You type a rough prompt such as "Write an email firing a client politely", pick an optimization target and a set of flags, and the compiler returns a prompt organized into `<system_role>`, `<task_boundaries>`, `<input_schema>` and `<output_format>` sections. A diagnostics tab reports how much ambiguity was removed, which variables were detected and which guardrails were added. A sandbox tab lets you fill in the variables and preview a mock run.

The app works immediately in Simulation Mode with no API key. Custom API Mode optionally sends your intent to OpenAI, Anthropic or Gemini for a live rewrite.

## Features

- **Simulation Mode:** a deterministic, pure JavaScript heuristic compiler with a 700ms loading state.
- **Custom API Mode:** live compilation through OpenAI, Anthropic or Gemini using your own key.
- **Optimization targets:** Production Agent / Tool Use, Structured JSON Extraction, Creative Direction, Academic Research.
- **Optimization flags:**
  - Inject strict XML delimiters
  - Enforce negative constraints (anti-hallucination)
  - Parameterize inputs (`{{variables}}`)
  - Include a chain-of-thought reasoning block
- **Output tabs:**
  - *Compiled System Prompt*: highlighted monospace view with a copy button and confirmation tooltip.
  - *Compiler Diagnostics*: ambiguity score and reduction, pass/fail lint matrix, detected variables, injected guardrails.
  - *Test Run Sandbox*: fill the detected variables, see the resolved prompt and a simulated output.
- **Responsive layout:** two columns on desktop, stacked on narrow tablet and phone widths.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | HTML shell that loads React 18, ReactDOM, Babel standalone, `styles.css` and `app.jsx` |
| `styles.css` | Dark slate/zinc theme, layout, tabs, tooltip and spinner styles |
| `app.jsx` | React application, heuristic compiler, diagnostics, sandbox and API clients |

## Running locally

Babel loads `app.jsx` with a network request, so opening `index.html` through `file://` will not work. Serve the folder over HTTP instead:

```bash
cd meta-prompt-optimizer
python3 -m http.server 8000
```

Then open `http://localhost:8000`. Any static server works, for example `npx serve`.

An internet connection is needed the first time to fetch React and Babel from the unpkg CDN.

## Usage

1. Enter your rough intent in the text area.
2. Choose an optimization target.
3. Toggle the flags you want.
4. Select **Compile Prompt**.
5. Copy the prompt from the first tab, review the audit in the second, and try it with sample values in the third.

### Custom API Mode

1. Switch the header toggle to **Custom API Mode**.
2. Choose a provider and paste your API key.
3. Compile as usual. If the request fails, the app shows an error and you can switch back to Simulation Mode.

The key is held in component state only. It is never written to storage, and it is sent only to the provider you selected. Requests are made directly from the browser, so use a key you can revoke and avoid this mode on shared machines.

Default models are set in the `MODELS` constant at the top of `app.jsx`. Update them if a provider retires a model.

## How the simulation compiler works

1. Extracts candidate variables with a small rule table (for example, "client" becomes `{{client_name}}`) and falls back to keywords from your text.
2. Builds each section from the chosen target and flags.
3. Scores ambiguity from input length and vague words, then estimates the reduction based on how many flags are on.
4. Lists guardrails based on the target and flags.

Diagnostics scores are rule-based estimates, not measurements of model behavior. The sandbox output is simulated and does not call any model.

## Customizing

- **Add a target:** add an entry to `TARGETS` in `app.jsx` with a `label`, `role`, `out` and `guard` list, and it appears in the dropdown.
- **Add variable rules:** add a `[regex, "variable_name"]` pair to `RULES`, and a description to `DESC`.
- **Change the theme:** edit the CSS variables at the top of `styles.css`.

## Limitations

- The heuristic compiler is template-based and does not understand your intent semantically.
- Live mode depends on provider CORS support and your own key and quota.
- Babel standalone compiles in the browser, which is fine for a tool like this but not ideal for large production deployments. For that, move the code into a bundler such as Vite.
