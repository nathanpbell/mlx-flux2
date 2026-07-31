# Vendored skill

Copied verbatim from Black Forest Labs' public skills repo — do not hand-edit;
re-vendor from upstream instead. `SOURCE.md` is the only file added locally.

- Upstream: https://github.com/black-forest-labs/skills/tree/master/skills/flux-best-practices
- Pinned commit: `d0793c84262183e8220a95fc1faeb5e2560aa6ef`
- Vendored: 2026-07-28
- Version: 1.0.0 (per upstream `metadata.json`)

## Layout

This follows the [Agent Skills](https://agentskills.io) open standard: a folder
containing `SKILL.md` with `name` + `description` frontmatter. The spec defines the
folder format, not where it lives — discovery paths are per-client, and
`.agents/skills/` is the vendor-neutral one.

`.agents/skills/flux-best-practices/` is the single canonical copy:

| Client                      | Discovery                                          |
| --------------------------- | -------------------------------------------------- |
| Codex                       | reads `.agents/skills/` natively (only path)        |
| Cursor                      | reads `.agents/skills/` natively                    |
| Gemini CLI                  | reads `.agents/skills/`; takes precedence over `.gemini/skills/` |
| Claude Code                 | reads `.claude/skills/` only → symlinked here       |

`.claude/skills/flux-best-practices` is a symlink to this directory. Claude Code
follows symlinks and loads a skill once even when the same target is reachable from
several locations, so the symlink adds no duplicate. Cursor also scans
`.claude/skills/` for compatibility and dedupes the same way.

Validate after any change with the spec's reference implementation:

```bash
npx skills-ref validate .agents/skills/flux-best-practices
```

### Not vendored

Upstream also ships an `AGENTS.md` — all 12 rules flattened into one 54KB file, for
agents that predate skill support. Every client targeted here reads the standard, so
vendoring it would just be a second copy of the same content to keep in sync. The
refresh script below can pull it if that ever changes.

## Refresh

Run from this directory:

```bash
BASE=https://raw.githubusercontent.com/black-forest-labs/skills/master/skills/flux-best-practices
curl -sfL "$BASE/SKILL.md" -o SKILL.md
curl -sfL "$BASE/metadata.json" -o metadata.json
# curl -sfL "$BASE/AGENTS.md" -o AGENTS.md   # flattened single-file variant; see above
for f in _sections core-principles flux1-models flux2-models hex-color-prompting \
         i2i-prompting json-structured-prompting model-selection-guide \
         multi-reference-editing negative-prompt-alternatives t2i-prompting typography-text; do
  curl -sfL "$BASE/rules/$f.md" -o "rules/$f.md"
done
```

## Notes

- `SKILL.md` mentions a companion **bfl-api** skill (BFL's hosted API: endpoints,
  polling, webhooks). That skill is not vendored here — this repo runs FLUX.2
  locally via MLX and does not call the BFL API.
- Model guidance covers the full hosted lineup (`max`, `pro`, `flex`, …). Only the
  open-weight klein/dev families are relevant to local generation here; the
  prompting rules themselves apply regardless.
