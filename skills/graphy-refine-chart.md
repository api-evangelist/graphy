---
name: Refine and annotate a chart with Graphy
description: Iterate on an existing Graphy GraphConfig — get suggestions, transform the data, restyle, narrate, and annotate — all with natural language.
api: openapi/graphy-agents-openapi.yaml
operations: [generateSuggestions, generateMutation, generateGraph, generateNarrative, generateAnnotations]
method: generated
generated: '2026-07-19'
---

# Refine and annotate a chart with Graphy

You already have a `GraphConfig` (from extract, from a suggestion, or hand-built). Use the agents to improve it.

## Auth & conventions
- Base URL `https://agents.graphy.dev`; `Authorization: Bearer graphy_...`; `Content-Type: application/json`.
- All responses are **SSE** streams; consume `progress`/`complete`/`error` events. Take the result from the `complete` event's `config`. See `conventions/graphy-conventions.yml`.

## Steps
1. **Get suggestions** — `POST /api/v0/suggestions` (`generateSuggestions`) with the `config` and a `userPrompt`. Returns `suggestions[]` (`dataPrepPrompt`, `chartType`, `summary`); optionally cap with `maxSuggestionCount`.
2. **Transform the dataset** — `POST /api/v0/mutate` (`generateMutation`) with a natural-language prompt to filter, group, aggregate, derive, or sort. Only `config.data` changes.
3. **Restyle the chart** — `POST /api/v0/generate` (`generateGraph`) with a prompt like "change to a stacked column chart and use the brand palette".
4. **Narrate** — `POST /api/v0/narrate` (`generateNarrative`) writes title/subtitle/caption into `config.content` (TipTap JSON). Requires `userPrompt`.
5. **Annotate** — `POST /api/v0/annotate` (`generateAnnotations`) to add highlights, tooltips, and stickers with natural language.

Each step returns an updated `GraphConfig`; feed it into the next. Persist/render with `@graphysdk/core`.
