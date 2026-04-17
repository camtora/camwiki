# wiki/decisions

Architecture decision records. Created and maintained by Claude.
Use template 6.4 from CLAUDE.md.

## Structure

ADRs are organized in one subdirectory per project or domain:

```
wiki/decisions/
  infra/          ← cross-cutting infrastructure decisions
  haymaker/       ← Haymaker-specific decisions
  rotosync/       ← Rotosync-specific decisions
  ...
```

Filenames inside subdirs do not need a `decision-` prefix — the subdir provides
the context. The `infra/` files keep their `decision-` prefix as a legacy
exception (they predate this convention).
