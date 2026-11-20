<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 EON contributors
-->

# Contributing to eon-download-manager-spec

The organisation-wide guide —
[`.github/CONTRIBUTING.md`](https://github.com/EON-Extensible-Open-Network/.github/blob/main/CONTRIBUTING.md)
— applies: Code of Conduct, DCO sign-off (`git commit -s`), Conventional Commits with the
`spec` scope.

- **A contract change comes with examples**: at least one file in `examples/valid` and one
  in `examples/invalid`, named `<handoff|reply|resume>-*.json`.
- **Patterns must mean the same in JavaScript and Rust.** Keep to plain character
  classes; avoid escapes inside classes.
- **Say what it costs:** who has to change, and what happens to those who do not.
- Contributions are licensed Apache-2.0.
