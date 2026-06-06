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

## Related Skills

- [hermes-agent-skill-authoring](https://github.com/search?q=hermes-agent-skill-authoring) — Author Skills from scratch
- [skill-marketplace](https://skills.sh) — Browse and install Skills from agentskills.io

## License

MIT
