# Wiki Log

Append-only. One entry per operation. Parseable with:
```sh
grep "^## \[" .claude/wiki/log.md | tail -10
```

---

## [2026-05-14] note | Wiki initialized
Wiki structure created. CLAUDE.md schema written. index.md and log.md bootstrapped.
Pages created: .claude/wiki/index.md, .claude/wiki/log.md, .claude/wiki/overview.md
Ready to ingest sources from .claude/raw/.

## [2026-05-14] note | Restructured into .claude/
Moved raw/ and wiki/ into .claude/ so the wiki can be installed in any project with .claude added to .gitignore.
CLAUDE.md remains at project root (auto-loaded by Claude Code).
