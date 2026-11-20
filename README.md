<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 EON contributors
-->

# EON Download Manager — contracts

What EON Download Manager agrees with the world about, written down so that anyone can
implement the other side: a browser extension, a script, another download manager.

| Contract | Document | Schema |
|---|---|---|
| Handing a download over from a browser (native messaging) | [docs/handoff.md](docs/handoff.md) | [handoff](schemas/handoff.v0.schema.json), [reply](schemas/reply.v0.schema.json) |
| The `.eondl` resume state beside a `.part` file | [docs/resume-state.md](docs/resume-state.md) | [resume](schemas/resume.v0.schema.json) |

**Version 0 is unstable.** Fields may change until `v1`; every change comes with valid and
invalid examples in `examples/` and a line in [CHANGELOG.md](CHANGELOG.md).

```
npm ci
npm run validate     # every example in examples/valid passes, every one in examples/invalid fails
```

The application: [eon-download-manager-app](https://github.com/EON-Extensible-Open-Network/eon-download-manager-app).
Plan: [madde 44](https://github.com/EON-Extensible-Open-Network/eon-docs) of the EON plan.

Licence: [Apache-2.0](LICENSE) — permissive on purpose, so other implementations are
welcome.
