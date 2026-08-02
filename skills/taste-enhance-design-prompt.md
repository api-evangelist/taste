---
name: Enhance a design prompt with brand guidelines
description: >-
  Use the Taste Engine API to turn a plain website- or slide-design prompt into a
  brand-consistent, enhanced prompt by pulling design guidelines extracted from a
  reference URL or a prior submission.
api: openapi/taste-taste-engine-openapi.json
operations:
  - enhance_prompt_enhance__post
  - enhance_slides_prompt_enhance_slides_post
  - liveness_check_health_get
---

# Enhance a design prompt with brand guidelines

The Taste Engine API extracts design guidelines and generates creative briefs. It
enhances a user's design prompt with brand-specific design tokens so downstream
generation stays on-brand.

## Auth
All calls require an `Authorization: Bearer <token>` header. Without it the
gateway returns `401 {"detail": "Authentication required..."}`. See
`authentication/taste-authentication.yml`.

## Website prompt flow
1. (Optional) Check liveness with `GET /health` (`liveness_check_health_get`).
2. Call `POST /enhance/` (`enhance_prompt_enhance__post`) with a JSON body:
   - `prompt` (required) — the design prompt, e.g. "Create a landing page for a SaaS product".
   - `url` (required) — a reference URL or submission ID whose brand guidelines to apply, e.g. `https://stripe.com`.
   - Optional: `strategy` (`full_rewrite` | `augment` | `structured` | `minimal`, default `structured`),
     `surface_type` (a `DesignSurface` enum value such as `landing_page`),
     `include_sections` / `exclude_sections`, `additional_context`,
     `auto_extract` (set `true` to trigger extraction if the URL has no guidelines yet).
3. Read `enhanced_prompt` from the `EnhanceResponse`; `structured_prompt`,
   `brand_context`, `reasoning`, and `model_used` explain the enhancement.

## Slides prompt flow
Call `POST /enhance/slides` (`enhance_slides_prompt_enhance_slides_post`) with:
- `prompt` (required) — e.g. "Create a pitch deck for our Series A fundraise".
- `submission_id` (required) — the prior slides extraction to pull guidelines from.
- Optional: `strategy`, `slide_surface` (a `SlideSurface` enum such as `pitch_deck`),
  `num_slides`, `include_slide_types` / `exclude_slide_types`, `additional_context`.

Both operations return the same `EnhanceResponse` shape.

## Errors
- `401` — missing/invalid bearer token.
- `422` — request body failed validation; `detail[]` lists `loc`/`msg`/`type` per field.
See `errors/taste-problem-types.yml`. Responses are not idempotent (no
Idempotency-Key contract) — see `conventions/taste-conventions.yml`.
