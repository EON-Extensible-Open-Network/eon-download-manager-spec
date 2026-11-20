<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 EON contributors
-->

# Changelog

## Unreleased

- The application registers the native host at every start rather than on request;
  *Disconnect* removes it for good. A `ping` is the extension saying hello: the
  application records which browser, so it can show the extension as connected.

## 0.0.0 — first contracts

- Browser hand-off over native messaging (`org.eon.download_manager`): `ping`, `download`,
  `links`, `media`, `refresh`; one message, one reply; only `http(s)` addresses; cookies
  only for the item's own address.
- Resume state `.eondl`: segments as `[start, pos, end)`, atomic writes at least once a
  second, `If-Range` validators, HLS progress.
