# API Documentation → AI Agent Skills

Convert any API documentation (OpenAPI, Swagger, Markdown, HTML, PDF) into **agentskills.io** standard Skill packages — ready to use with Hermes Agent, Claude Code, Codex, or Cursor.

## What Is This?

A practical, battle-tested workflow for turning API docs into structured AI Agent Skills. Each Skill includes:

- **SKILL.md** — 150–250 line reference doc with frontmatter, endpoint tables, workflows, examples
- **scripts/** — `healthcheck.py`, `example_call.py`, `poll_task.py` (for async APIs)
- **references/** — `api-summary.md`, `endpoint-index.md`

Verified at scale: **11 Skills (73 files)** generated from a single bootstrap script.

## When to Use

- Turn an API into a Skill ("make a Skill for model X")
- Batch-generate Skills for multiple models from one API gateway
- Upgrade existing Skills after API documentation changes

## Quick Start

```bash
# 1. Discover API endpoints
curl -s "https://YOUR-SITE.apifox.cn/llms.txt" > llms.txt

# 2. Fetch endpoint specs
curl -s "https://YOUR-SITE.apifox.cn/api-XXXXXXX.md" > endpoint.md

# 3. Build API_REFERENCE.md with all endpoints

# 4. Have Codex generate Skills from the spec

# 5. Verify
python3 -m py_compile $(find skills -path '*/scripts/*.py' | sort)
grep -r "{ENV_VAR}" skills/ || echo "No placeholders found ✓"
```

## Workflow

```
1. Research    → Discover API endpoints (llms.txt, OpenAPI, docs)
2. Reference   → Build API_REFERENCE.md with all endpoint details
3. Design      → Create DESIGN.md with Skill structure and data models
4. Implement   → Codex generates Skills from spec (batch via bootstrap script)
5. Review      → UX review from Agent perspective (4/10 → 8/10 target)
6. Optimize    → Fix P0/P1 issues (description, exit codes, dry-run, polling)
7. Verify      → Syntax check, placeholder check, dry-run test, install
```

## Generated Skill Structure

```
skill-name/
├── SKILL.md              # Main skill doc (frontmatter + 11 sections)
├── scripts/
│   ├── healthcheck.py    # Env var + connectivity check (--dry-run/--live)
│   ├── example_call.py   # Minimal working example (--dry-run/--live)
│   └── poll_task.py      # Async polling (for async-only skills)
└── references/
    ├── api-summary.md    # Capability overview, model list, workflow
    └── endpoint-index.md # All endpoints with method, path, params
```

## Supported API Patterns

| Pattern | Examples | Auth |
|---------|----------|------|
| OpenAI-compatible | Chat, Image, Audio, Embeddings | `Authorization: Bearer` |
| Gemini native | generateContent, embedContent | `?key=` query param |
| Async task | Video, some Image (submit → poll → result) | Varies |
| Platform-specific | Kling, Replicate, Fal.ai, MINIMAX, VIDU | Bearer token |

## Reference: CursorAI API Gateway Coverage

This workflow was validated against the [CursorAI API Gateway](https://cursorai.apifox.cn/) which aggregates 20+ providers:

- **Chat**: GPT-5.x, Claude 4, Gemini 2.5/3, DeepSeek, Qwen
- **Responses API**: o3-pro, codex-mini (OpenAI new format)
- **Image**: GPT Image 1/1.5/2, DALL·E 3, Gemini Image, Grok Image, Midjourney, Ideogram, FLUX, 即梦, 豆包 seedream, 千问, 万向 wan, 腾讯AIGC
- **Video**: Sora, Veo 3/3.1, Grok, Kling, Luma, Runway, 即梦, 海螺, 豆包 seedance, 通义万象, TC-Vidu, 腾讯AIGC, omni, Fal.ai, Replicate, MINIMAX, VIDU, 阿里pix
- **Audio**: OpenAI TTS/STT (incl. gpt-4o-mini-tts, gpt-4o-transcribe), Gemini TTS, MINIMAX TTS, VIDU TTS, 通义万象 TTS, 可灵 TTS
- **Music**: Suno (inspiration, custom, extend, splice modes)
- **Embeddings**: OpenAI, Gemini
- **Rerank**: Reranking models
- **Platforms**: 可灵 Kling (full platform), Replicate, Fal.ai, MINIMAX, VIDU, 阿里 pix

## Quality Checklist

After generation, verify each Skill against these dimensions:

| Dimension | Check |
|-----------|-------|
| Frontmatter | description starts with "Use when...", no unreplaced placeholders |
| Scripts | Return non-zero on errors, support `--dry-run` mode |
| Async | Has `poll_task.py` with timeout, backoff, terminal state handling |
| Cross-skill | Consistent base URL, auth scheme, file structure |
| Content | Real JSON examples (not placeholders), skill-specific pitfalls |

Full checklist: [references/quality-checklist.md](references/quality-checklist.md)

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Codex generates duplicate template content | Explicitly instruct "write Skill-specific content only" |
| Scripts return 0 on errors | Enforce non-zero for all errors except HTTP 2xx |
| Missing dry-run mode | Add `--dry-run` (default) / `--live` to all scripts |
| No async polling | Include `poll_task.py` with backoff and terminal states |
| Placeholder variables | `grep -r "{ENV_VAR}" skills/` after generation |
| Mixed auth patterns | Document exact auth method per endpoint (Bearer vs query param) |
| Multiple format variants | Each format needs its own endpoint documentation |
| Video model version drift | Pin model versions, note update date |

## Verification

```bash
# Syntax check all scripts
python3 -m py_compile $(find skills -path '*/scripts/*.py' -not -path '*__pycache__*' | sort)

# Check for unreplaced placeholders
grep -r "{ENV_VAR}" skills/ || echo "No placeholders"

# Check for duplicate template content
grep -c "Note [0-9]" skills/*/SKILL.md | grep -v ":0$" || echo "No duplicate notes"

# Verify all descriptions follow format
grep -L "Use when" skills/*/SKILL.md || echo "All descriptions correct"
```

## Changelog

### v2.0.0 (2026-06)
- Added Responses API coverage (o3-pro, codex-mini)
- Added Sora video generation
- Added 豆包 (seedance, doubao-2.0), 通义万象, TC-Vidu, 腾讯AIGC, omni video
- Added 即梦, 海螺 (MiniMax) video
- Added GPT Image 1.5, GPT Image 2, Grok Image
- Added 即梦绘画, 豆包 seedream, 千问 Qwen-Image, 万向 wan image models
- Added Replicate aggregation platform (FLUX, Stable Diffusion, Imagen 4, etc.)
- Added Fal.ai aggregation platform (Veo3, Kling video, Seedream, etc.)
- Added MINIMAX official (TTS, video, voice clone)
- Added VIDU official (video, image, audio)
- Added 阿里 pix platform (video generation)
- Added 可灵 Kling standalone platform (full feature set)
- Added Rerank models
- Added GPT-4o-mini-tts, gpt-4o-transcribe
- Added DeepSeek OCR, Qwen MT Turbo
- Documented 5 API patterns (OpenAI-compat, Gemini native, async task, platform-specific, Suno)
- Added 3 new pitfalls (auth patterns, format variants, version drift)

### v1.0.0
- Initial release with ChatGPT, Claude, Gemini, Image, Video, Audio, Music, Embeddings

## Related Skills

- [hermes-agent-skill-authoring](https://github.com/search?q=hermes-agent-skill-authoring) — Author Skills from scratch
- [skill-marketplace](https://skills.sh) — Browse and install Skills from agentskills.io

## License

MIT
