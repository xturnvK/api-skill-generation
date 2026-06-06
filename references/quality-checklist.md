# Skill Quality Review Dimensions

Used when reviewing AI-generated Skills from an Hermes Agent user perspective.

## 1. SKILL.md Quality

### Frontmatter
- description starts with "Use when..." and describes trigger class
- name is lowercase-hyphens, ≤64 chars
- All metadata fields present (version, author, license, required_env)
- No unreplaced placeholders ({ENV_VAR}, {model}, {BASE_URL})

### Content Structure
- Overview explains capability boundary (what it can/can't do)
- When to Use has positive AND negative triggers
- Authentication section is specific (Bearer vs API key vs query param)
- Endpoint table has: method, path, parameters, response format, sync/async, risk
- Workflows are task-specific, not generic steps
- Request/response examples are real JSON objects
- Parameter guide has decision criteria (when to use which model/size/format)
- Error handling lists specific HTTP codes and recovery actions
- Pitfalls are Skill-specific, not cross-copied

## 2. Scripts Quality

### healthcheck.py
- Returns 2 on missing env var
- Returns 1 on network failure (NOT 0)
- Returns 1 on non-2xx response
- Supports --dry-run (default) / --live
- Uses python3 (not python)

### example_call.py
- Supports --dry-run (default) / --live
- Only exits 0 on HTTP 2xx
- Redacts API key in output (Bearer <redacted>)
- Covers main use case of the Skill
- Uses python3

### poll_task.py (async Skills only)
- Implements submit → poll → result flow
- Has timeout, initial delay, max delay (backoff)
- Handles terminal states (completed, failed, pending, processing)
- Saves result to file
- Supports --dry-run

## 3. References Quality

### api-summary.md
- Lists provider, base URL, auth scheme
- Lists all supported models
- Describes primary workflow
- Shows minimal payload example

### endpoint-index.md
- Lists all endpoints with method, path, parameters
- Indicates sync vs async
- Notes risk level for destructive operations

## 4. Cross-Skill Consistency

- Base URL consistent across all Skills
- Auth scheme consistent (check if Bearer vs API key vs query param)
- File structure identical (SKILL.md + scripts/ + references/)
- No overlapping Skills (each covers distinct model/provider)

## 5. API Pattern Coverage

For multi-provider gateways (like CursorAI), verify:

- OpenAI-compatible endpoints documented separately from native format
- Gemini native (`:generateContent`) vs chat-compatible (`/v1/chat/completions`) not merged
- Async task endpoints include submit + poll + result (not just submit)
- Platform-specific auth (Bearer vs query param) documented per-endpoint
- Model version pinned in examples (e.g., `sora-2`, `veo-3.1-generate-preview`)

## 6. Scoring Guide

| Score | Criteria |
|-------|----------|
| 1-3   | Directory structure only, no real content |
| 4-5   | Has content but template-heavy, scripts broken |
| 6-7   | Skill-specific content, scripts work, minor gaps |
| 8-9   | Comprehensive, all scripts verified, real examples |
| 10    | Production-ready, edge cases covered, cross-tested |
