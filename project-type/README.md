# Project-type conventions (language-agnostic)

Cross-cutting rules for the two kinds of thing Codechu builds.
Language-specific extensions live in [`lang/`](../lang/).

| Project type | Definition | Conventions |
|---|---|---|
| **Library** | Reusable component published to a package registry, consumed by other software | [LIBRARY.md](LIBRARY.md) |
| **Application** | End-user product installed and run by a human | [APPLICATION.md](APPLICATION.md) |
| **Proprietary library** | Closed-source / source-available library sold to customers | [PROPRIETARY-LIBRARY.md](PROPRIETARY-LIBRARY.md) |
| **Proprietary application** | Closed-source / source-available end-user application | [PROPRIETARY-APPLICATION.md](PROPRIETARY-APPLICATION.md) |

These extend [`STANDARDS.md`](../STANDARDS.md). When a rule here
disagrees with a language-specific file under `lang/`, the
language-specific file wins for that language. The
`PROPRIETARY-*.md` files extend (never weaken) their OSS
counterparts.

## Where things land

```
codechu-org/
├── STANDARDS.md                       # all products, all languages
├── TEMPLATE-CHECKLIST.md              # generic bootstrap
├── project-type/
│   ├── LIBRARY.md                     # any library (OSS-friendly)
│   ├── APPLICATION.md                 # any app (OSS-friendly)
│   ├── PROPRIETARY-LIBRARY.md         # closed-source library extras
│   └── PROPRIETARY-APPLICATION.md     # closed-source app extras
└── lang/
    └── python/
        ├── LIBRARY.md                 # Python library specifics
        ├── APPLICATION.md             # Python app specifics
        └── PROPRIETARY.md             # Python closed-source extras
```

Read in order: STANDARDS → project-type → lang. Each layer narrows.
