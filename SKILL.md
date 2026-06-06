---
name: api-skill-generation
description: "Use when converting API documentation into agentskills.io Skill packages for Hermes, Claude Code, Codex, or Cursor."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [api, skills, generation, documentation, cursorai, openapi]
    related_skills: [hermes-agent-skill-authoring, codex]
---

# API Documentation → AI Agent Skills

Convert API documentation (OpenAPI, Markdown, HTML, PDF) into agentskills.io
standard Skill packages that any compatible Agent can load and use.

## When to Use

- User asks to "turn this API into a Skill" or "make a Skill for model X"
- Converting CursorAI, OpenAI, Anthropic, Google, or other API docs into Skills
- Batch-generating Skills for multiple models from a single API gateway
- Upgrading existing Skills after API documentation changes

## When NOT to Use

- For authoring Skills from scratch (use `hermes-agent-skill-authoring` instead)
- For non-API knowledge Skills (coding patterns, debugging guides, etc.)

## Workflow Overview

```
1. Research    → Discover API endpoints (llms.txt, OpenAPI, docs)
2. Reference   → Build API_REFERENCE.md with all endpoint details
3. Design      → Create DESIGN.md with Skill structure and data models
4. Implement   → Codex generates Skills from spec (batch via bootstrap script)
5. Review      → UX review from Agent perspective (4/10 → 8/10 target)
6. Optimize    → Fix P0/P1 issues (description, exit codes, dry-run, polling)
7. Verify      → Syntax check, placeholder check, dry-run test, install
```

## Step 1: API Discovery

### llms.txt Pattern (Fastest)
Many Apifox-hosted doc sites expose a structured index:
```bash
curl -s "https://SITE.apifox.cn/llms.txt" > llms.txt
# Filter by category
grep -E "^- .* > (聊天|绘画|视频|音频|音乐)" llms.txt
# Fetch individual endpoint specs (contain OpenAPI YAML)
curl -s "https://SITE.apifox.cn/api-XXXXXXX.md" > endpoint.md
```

### OpenAPI Spec
If the API publishes an OpenAPI/Swagger spec, fetch and parse it directly.

### Manual Browsing
Browse the documentation site, extract endpoints, auth format, and examples.

## Step 2: Build API Reference

Write `API_REFERENCE.md` covering:
- Base URL and auth scheme
- All endpoints grouped by capability (chat, image, video, audio, etc.)
- For each endpoint: path, method, parameters, request/response format
- Model variants and their differences
- Async task patterns (submit → poll → result)
- Error codes and recovery

## Step 3: Implement Skills

### Batch Generation via Bootstrap Script
For generating many similar Skills, have Codex write a Python generator
script first, then run it. The script encodes template logic and produces
all files in one execution. Verified: 11 Skills (73 files) generated via
a single bootstrap script (~86K tokens).

### Per-Skill Structure
```
skill-name/
  SKILL.md                    # 150-250 lines, frontmatter + 9 sections
  scripts/
    healthcheck.py            # Env var + connectivity check, --dry-run/--live
    example_call.py           # Minimal working example, --dry-run/--live
    poll_task.py              # Async polling (for async-only Skills)
  references/
    api-summary.md            # Capability overview, model list, workflow
    endpoint-index.md         # All endpoints with method, path, params
```

### SKILL.md Template Structure
1. Frontmatter (name, description "Use when...", version, metadata)
2. Overview (one paragraph + supported models + core features)
3. When to Use (3-5 positive + 3-5 negative triggers)
4. Authentication Configuration
5. API Endpoint Quick Reference (table: method, path, params, response, sync/async, risk)
6. Typical Workflow (numbered steps)
7. Request And Response Examples (real JSON, not placeholders)
8. Parameter Selection Guide (decision table)
9. Error Handling (HTTP codes + recovery)
10. Common Pitfalls (Skill-specific)
11. Verification Checklist

## Step 4: Quality Review

After Codex generates Skills, review from Hermes Agent perspective:
- Load each Skill and check if the Agent can immediately start working
- Verify description starts with "Use when..."
- Verify no duplicate template content
- Verify scripts compile and handle errors correctly
- Run healthcheck.py and example_call.py in dry-run mode
- Check that async Skills have poll_task.py

See `references/quality-checklist.md` for the full review dimensions.

## Step 5: Optimize

Fix issues by priority:
- **P0**: Description format, placeholder replacement, exit codes, dry-run
- **P1**: Async polling scripts, endpoint tables, parameter guides, examples
- **P2**: Model coverage gaps, cross-skill consistency, reference enrichment

## CursorAI API Gateway

The CursorAI API proxy (`https://api.cursorai.art`) aggregates multiple
AI model providers under a single endpoint with OpenAI-compatible protocol.
Useful for generating Skills covering many model types from one source.

### Covered Categories
- Chat: GPT-5.5, Claude, Gemini, DeepSeek
- Image: GPT Image, Gemini Image, Midjourney, FLUX, DALL-E, Ideogram
- Video: Grok, Veo, Luma, Runway, MiniMax, Kling
- Audio: OpenAI TTS/STT, Gemini TTS
- Embeddings: OpenAI, Gemini
- Music: Suno

### API Reference
- Base URL: `https://api.cursorai.art`
- Auth: `Authorization: Bearer $CURSORAI_API_KEY`
- Full endpoint catalog: `https://cursorai.apifox.cn/llms.txt`

## Common Pitfalls

1. **Codex generates duplicate template content**: When batch-generating Skills,
   Codex often produces identical "Operational Notes" sections across all Skills.
   Fix: include explicit instructions to write Skill-specific content only, and
   run `grep -c "Note [0-9]" skills/*/SKILL.md` to detect duplicates.

2. **Scripts return 0 on errors**: Codex-generated healthcheck scripts often
   catch network exceptions and return 0 (success). Fix: enforce that only
   HTTP 2xx returns 0; all errors return non-zero.

3. **Missing dry-run mode**: Agents can't test Skills without API keys if
   scripts only work in live mode. Fix: add `--dry-run` (default) / `--live`
   argument parsing to all scripts.

4. **No async polling**: Media generation APIs are async (submit → poll → result).
   Skills for these APIs must include poll_task.py with timeout, backoff,
   and terminal state handling.

5. **Placeholder variables**: Codex may leave `{ENV_VAR}`, `{model}`, or
   `{BASE_URL}` in generated files. Fix: `grep -r "{ENV_VAR}" skills/` after
   generation.

## Verification

```bash
# Full verification suite
python3 -m py_compile $(find skills -path '*/scripts/*.py' -not -path '*__pycache__*' | sort)
grep -r "{ENV_VAR}" skills/ || echo "No placeholders"
grep -c "Note [0-9]" skills/*/SKILL.md | grep -v ":0$" || echo "No duplicate notes"
grep -L "Use when" skills/*/SKILL.md || echo "All descriptions correct"
```
