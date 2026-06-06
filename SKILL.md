---
name: api-skill-generation
description: "Use when converting API documentation into agentskills.io Skill packages for Hermes, Claude Code, Codex, or Cursor."
version: 2.0.0
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

#### Chat (聊天)
| Provider | Format | Models |
|----------|--------|--------|
| OpenAI | `/v1/chat/completions` | GPT-5.x, GPT-4o, o1, o3, o4-mini |
| Anthropic Claude | `/v1/chat/completions` + native | Claude 4, Claude 3.5 |
| Google Gemini | `/v1beta/models/*:generateContent` | Gemini 2.5 Pro/Flash, Gemini 3 Pro |
| DeepSeek | `/v1/chat/completions` | deepseek-v3.1, deepseek-ocr |
| Qwen | `/v1/chat/completions` | qwen-mt-turbo |

#### Responses API (新)
| Endpoint | Models |
|----------|--------|
| `/v1/responses` | o3-pro, codex-mini-latest, gpt-5 |

#### Image Generation (绘画)
| Provider | Endpoint Pattern | Models |
|----------|-----------------|--------|
| GPT Image | `/v1/images/generations` | gpt-image-1, gpt-image-1.5, gpt-image-2 |
| DALL·E | `/v1/images/generations` | dall-e-3 |
| Gemini Image | `:generateContent` (native) | gemini-2.5-flash-image, gemini-3-pro-image |
| Grok Image | `/v1/images/generations` | grok-image |
| Midjourney | `/mj/submit/imagine` | v6.1, v6 |
| Ideogram | `/ideogram/v1/*` | ideogram-v3 |
| FLUX | Replicate + Fal.ai formats | flux-kontext-dev/pro/max, flux-1.1-pro-ultra |
| 即梦绘画 | `/v1/images/generations` | jimeng |
| 豆包 Image | `/v1/images/generations` | doubao-seedream-4.0/4.5 |
| 千问 Image | `/v1/images/generations` | qwen-image-max, z-image-turbo |
| 万向 wan | `/v1/images/generations` | wan2.7-image-pro |
| 腾讯AIGC | async task API |混元 |
| Fal.ai | `/fal-ai/*` | nano-banana, flux-1/dev |
| Replicate | `/replicate/v1/models/*` | flux-kontext, stable-diffusion, imagen-4 |

#### Video Generation (视频)
| Provider | Endpoint | Models |
|----------|----------|--------|
| Sora | `/v1/video/create` | sora-2, sora-2-pro |
| Veo | `/v1/video/create` | veo-3, veo-3.1 |
| Grok | `/v1/video/create` | grok-video-3 |
| Kling | `/kling/v1/videos/*` | kling-v2.5 |
| Luma | `/v1/video/create` | luma |
| Runway | `/v1/video/create` | runway |
| 即梦 | `/v1/video/create` | jimeng-video |
| 海螺 | `/v1/video/create` | minimax-video |
| 豆包 | `/v1/video/create` | seedance-1.5-pro, doubao-2.0 |
| 通义万象 | `/v1/video/create` | happyhorse-1.0 |
| TC-Vidu | `/v1/video/create` | vidu |
| 腾讯AIGC | async task API |混元视频 |
| omni | `/v1/video/create` | omni-video |
| Fal.ai | `/fal-ai/*` | veo3, kling-video |
| Replicate | `/replicate/v1/models/*` | minimax/video-01 |
| MINIMAX官方 | `/v1/video/create` | minimax-official |
| VIDU官方 | `/v1/video/create` | vidu-official |
| 阿里pix | `/v1/video/create` | pix-video |

#### Audio / Speech (语音)
| Provider | Endpoint | Models |
|----------|----------|--------|
| OpenAI TTS | `/v1/audio/speech` | tts-1, tts-1-hd, gpt-4o-mini-tts |
| OpenAI STT | `/v1/audio/transcriptions` | whisper-1, gpt-4o-transcribe |
| Gemini TTS | native `:generateContent` | gemini-tts |
| MINIMAX TTS | `/v1/t2a_v2` | minimax-tts |
| VIDU TTS | `/v1/audio/create` | vidu-tts |
| 通义万象 TTS | `/v1/audio/create` | qwen-tts |
| 可灵 TTS | `/kling/v1/tts` | kling-tts |

#### Music (音乐)
| Provider | Endpoint | Models |
|----------|----------|--------|
| Suno | `/suno/submit/music` | chirp-v3-0, chirp-v3-5 |

#### Embeddings (向量化)
| Provider | Endpoint | Models |
|----------|----------|--------|
| OpenAI | `/v1/embeddings` | text-embedding-3-small/large |
| Gemini | native `:embedContent` | text-embedding-004 |

#### Rerank (重排序)
| Provider | Endpoint |
|----------|----------|
| Rerank | `/v1/rerank` |

#### Standalone Platforms (独立平台)
| Platform | Capabilities | Auth Pattern |
|----------|-------------|--------------|
| 可灵 Kling | video, image, audio, digital human, lip-sync | Bearer token |
| Replicate | image, video, audio via model version | Bearer token |
| Fal.ai | image, video via model path | API key |
| MINIMAX | TTS, video, voice clone | Bearer token |
| VIDU | video, image, audio | Bearer token |
| 阿里 pix | video generation | API key |

### API Reference
- Base URL: `https://api.cursorai.art`
- Auth: `Authorization: Bearer $CURSORAI_API_KEY`
- Full endpoint catalog: `https://cursorai.apifox.cn/llms.txt`
- Last updated: 2026-06 (based on cursorai.apifox.cn documentation)

## API Patterns

### Pattern 1: OpenAI-Compatible (Chat, Image, Audio)
```
POST /v1/chat/completions
POST /v1/images/generations
POST /v1/audio/speech
POST /v1/audio/transcriptions
POST /v1/embeddings
Authorization: Bearer $API_KEY
```

### Pattern 2: Gemini Native
```
POST /v1beta/models/{model}:generateContent
POST /v1beta/models/{model}:embedContent
?key=$API_KEY (query param)
```

### Pattern 3: Async Task (Video, some Image)
```
POST /v1/video/create          → { task_id }
GET  /v1/video/query/{task_id} → { status, result_url }
```
Status flow: pending → processing → completed | failed

### Pattern 4: Platform-Specific (Kling, Replicate, Fal.ai)
```
POST /kling/v1/videos/text2video    → { data: { task_id } }
GET  /kling/v1/videos/text2video/{id}
POST /replicate/v1/models/{org}/{model}/predictions → { id }
GET  /replicate/v1/predictions/{id}
POST /fal-ai/{model}              → { request_id }
GET  /fal-ai/{model}/requests/{request_id}
```

### Pattern 5: Suno Music
```
POST /suno/submit/music     → { data: { task_id } }
GET  /suno/query/{task_id}  → { status, audio_url }
```

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

6. **Mixed auth patterns**: Some endpoints use `Authorization: Bearer`, others
   use `?key=` query param (Gemini native). Document the exact auth method
   per endpoint, not just one generic pattern.

7. **Multiple format variants**: Many providers expose both OpenAI-compatible
   and native formats (e.g., Gemini chat-compatible vs native generateContent).
   Each format needs its own endpoint documentation — don't merge them.

8. **Video model version drift**: Video models (Sora, Veo, Kling) update
   frequently. Pin model versions in examples and note the update date.

## Verification

```bash
# Full verification suite
python3 -m py_compile $(find skills -path '*/scripts/*.py' -not -path '*__pycache__*' | sort)
grep -r "{ENV_VAR}" skills/ || echo "No placeholders"
grep -c "Note [0-9]" skills/*/SKILL.md | grep -v ":0$" || echo "No duplicate notes"
grep -L "Use when" skills/*/SKILL.md || echo "All descriptions correct"
```
