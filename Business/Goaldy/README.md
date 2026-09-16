# Goaldy — docs moved

**Created:** 2026-09-16
**Last Updated:** 2026-09-16

The Goaldy **PRD** and **Tech Design Document** no longer live in this vault.
They moved into the Goaldy repository on 2026-09-16 so that a feature and the
documentation describing it land in the same pull request:

| Document | New location |
|---|---|
| Product Requirements Document | `nisan-tagar/goaldy` → `docs/project/PRD.md` |
| Tech Design Document | `nisan-tagar/goaldy` → `docs/project/TDD.md` |
| Specs & phased implementation plans | `nisan-tagar/goaldy` → `docs/superpowers/` |

Their history up to the move is still in this repository's git log under the
old filenames:

```
git log --follow -- "Business/Goaldy/Goaldy — Product Requirements Document.md"
git log --follow -- "Business/Goaldy/Goaldy - Tech Design Document.md"
```

**Still here:** the Goaldy landing page, at `docs/goaldy/` in this repo
(`index.html`, `dashboard.jpg`, `logo.webp`). Its copy traces back to PRD §F14,
which now lives in the other repository — update §F14 first, then the page.
