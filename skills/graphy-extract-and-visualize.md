---
name: Extract data and build a chart with Graphy
description: Turn unstructured text, a PDF, an image, or a spreadsheet into a presentation-ready chart using the Graphy AI Agents API.
api: openapi/graphy-agents-openapi.yaml
operations: [extractFromProse, evaluateDataset, generateGraph, generateNarrative]
method: generated
generated: '2026-07-19'
---

# Extract data and build a chart with Graphy

Use the Graphy AI Agents API to go from raw source material to a finished, narrated chart.

## Auth & conventions
- Base URL: `https://agents.graphy.dev`
- Header: `Authorization: Bearer graphy_...` and `Content-Type: application/json` (see `authentication/graphy-authentication.yml`).
- Every endpoint streams **Server-Sent Events** (`text/event-stream`) with `progress`, `complete`, and `error` events. Read the stream to completion and use the `complete` event's `config`.
- Errors: `{ error, code?, retryable? }`. Retry only when `retryable: true` (`RATE_LIMIT_ERROR`, `PROCESSING_ERROR`, `TIMEOUT_ERROR`). See `errors/graphy-problem-types.yml`.

## Steps
1. **Extract a dataset** — `POST /api/v0/extract` (`extractFromProse`). Send `sourceText` and/or `attachments[]` (base64 with `mimeType`/`filename`). The `complete` event returns a `config` (a `GraphConfig` with a populated `data` object) plus `extractMeta` (confidence, warnings, needsUserInput). If `needsUserInput` is true, gather clarification before continuing.
2. **Pick the best chart type (optional, deterministic)** — `POST /api/v0/evaluate` (`evaluateDataset`) with the extracted `config`. It returns a `ranking[]` of `ChartFitEntry` (rank/type/score/verdict) using a rule-based scorer — no LLM, no tokens. Set `config.type` to the top-ranked type.
3. **Generate the chart** — `POST /api/v0/generate` (`generateGraph`) with the `config` and a `userPrompt` (e.g. "make this a clean column chart with a trend line"). Use the returned `config`.
4. **Add a narrative (optional)** — `POST /api/v0/narrate` (`generateNarrative`) with a `userPrompt`; the title/subtitle/caption are written into `config.content` as TipTap JSON.

Render the final `GraphConfig` with the `@graphysdk/core` charting components (see `components/graphy-components.yml`).
