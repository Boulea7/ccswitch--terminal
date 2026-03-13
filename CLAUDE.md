# ccswitch--terminal - Codebase Guide for AI Tools

## Project Overview

Single-file Python CLI (`ccsw.py`) for switching API providers across Claude Code, Codex CLI, and Gemini CLI.

## Key Files

| File | Purpose |
|------|---------|
| `ccsw.py` | Main script, stdlib only, ~350 lines |
| `bootstrap.sh` | Shell install: registers `ccsw` + `ccswitch` aliases |
| `~/.ccswitch/providers.json` | Provider config store (created at runtime) |
| `~/.ccswitch/active.env` | Active Gemini env vars (sourced by shell) |

## Architecture

```
main()
  load_local_env()       # reads .env.local into os.environ
  load_store()           # reads ~/.ccswitch/providers.json
  ensure_defaults()      # seeds built-in providers (88code, zhipu)
  cmd_switch / cmd_add / cmd_list / cmd_show / cmd_remove / cmd_alias_add
```

## Stdout / Stderr Contract

- `info()` -> **stderr only** (status messages)
- `emit_env()` -> **stdout only** (`export KEY='val'`)

This separation enables: `eval "$(ccsw gemini provider)"` - the export goes to stdout, captured by command substitution; status info goes to stderr, shown in terminal.

## providers.json Schema

```json
{
  "version": 1,
  "active": { "claude": "88code", "codex": "88code", "gemini": null },
  "aliases": { "88": "88code", "glm": "zhipu" },
  "providers": {
    "88code": {
      "claude": { "base_url": "...", "token": "$ENV_VAR", "extra_env": {} },
      "codex":  { "base_url": "...", "token": "$ENV_VAR" },
      "gemini": null
    }
  }
}
```

## Token Resolution

Tokens stored as `$MY_ENV_VAR` are resolved at switch time via `resolve_token()`. Literal strings are used as-is. This keeps secrets out of the JSON file.

## Config Write Targets

| Tool | File | Keys written |
|------|------|-------------|
| Claude Code | `~/.claude/settings.json` → `env` | `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_BASE_URL`, extra_env |
| Codex CLI | `~/.codex/auth.json` | `OPENAI_API_KEY`, `OPENAI_BASE_URL` |
| Gemini CLI | `~/.gemini/settings.json` | `security.auth.selectedType` |
| Gemini env | stdout + `~/.ccswitch/active.env` | `GEMINI_API_KEY` |

## Adding a New Built-in Provider

Edit `BUILTIN_PROVIDERS` dict in `ccsw.py`. Schema matches the providers.json structure above.
