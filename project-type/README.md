# Project-type conventions (language-agnostic)

Cross-cutting rules for the two kinds of thing Codechu builds.
Language-specific extensions live in [`lang/`](../lang/).

| Project type | Definition | Conventions |
|---|---|---|
| **Library** | Reusable component published to a package registry, consumed by other software | [LIBRARY.md](LIBRARY.md) |
| **Application** | End-user product installed and run by a human | [APPLICATION.md](APPLICATION.md) |

These extend [`STANDARDS.md`](../STANDARDS.md). When a rule here
disagrees with a language-specific file under `lang/`, the
language-specific file wins for that language.

## Where things land

```
codechu-org/
├── STANDARDS.md                   # all products, all languages
├── TEMPLATE-CHECKLIST.md          # generic bootstrap
├── project-type/
│   ├── LIBRARY.md                 # any library
│   └── APPLICATION.md             # any app
└── lang/
    └── python/
        ├── LIBRARY.md             # Python library specifics
        └── APPLICATION.md         # Python app specifics
```

Read in order: STANDARDS → project-type → lang. Each layer narrows.
