<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 EON contributors
-->

# Browser hand-off (v0)

How a browser extension — or any program — gives EON Download Manager something to
download. Schema: [`handoff.v0.schema.json`](../schemas/handoff.v0.schema.json); reply:
[`reply.v0.schema.json`](../schemas/reply.v0.schema.json).

## Transport: native messaging

The host is named **`org.eon.download_manager`**. Its manifest points at the
`eon-download-manager` executable itself; there is no separate helper program.

| Browser | Where the host manifest is registered (per user) |
|---|---|
| Chrome | Windows: `HKCU\Software\Google\Chrome\NativeMessagingHosts\org.eon.download_manager`; Linux: `~/.config/google-chrome/NativeMessagingHosts/`; macOS: `~/Library/Application Support/Google/Chrome/NativeMessagingHosts/` |
| Edge | Windows: `HKCU\Software\Microsoft\Edge\NativeMessagingHosts\…`; Linux: `~/.config/microsoft-edge/NativeMessagingHosts/`; macOS: `~/Library/Application Support/Microsoft Edge/NativeMessagingHosts/` |
| Chromium, Brave, Vivaldi | the Chromium locations for each |
| Firefox | Windows: `HKCU\Software\Mozilla\NativeMessagingHosts\…`; Linux: `~/.mozilla/native-messaging-hosts/`; macOS: `~/Library/Application Support/Mozilla/NativeMessagingHosts/` |

The application registers itself **per user, at every start** (no administrator rights;
renewing it keeps the path right if the program moves), and *Disconnect* removes the
registration and stops it from coming back. Chromium manifests list the
extension's origin in `allowed_origins`; Firefox manifests list its id in
`allowed_extensions`. Nothing else can start the host.

**Framing**, as the browsers define it: each message is a 32-bit length in the machine's
native byte order followed by that many bytes of UTF-8 JSON. The browser starts the
executable with the caller's origin (`chrome-extension://<id>/`) or, for Firefox, the
manifest path and the extension id as arguments; that is how the executable knows it is
running as a host.

**One message, one reply.** The extension uses `sendNativeMessage`. The host reads one
message (at most **8 MiB**; larger is answered with `too_large`), checks it against the
schema, hands it to the application window — starting it if it is not running — and
writes one reply, then exits. The host never downloads anything itself and never answers
with data about other downloads.

**`ok: true` means the window has it.** The host answers only once the running window
has taken the hand-off. If it does not within a short time (a cold start of the window
included), the host withdraws it and answers `unavailable`, and the extension leaves the
download to the browser. A download is never cancelled in the browser on the strength of
a hand-off nobody received. Nothing may rely on a process the host starts outliving it:
a browser may end every such process when the host exits (Firefox does, on Windows).

## Messages

`kind` says what the person did:

| kind | When | items |
|---|---|---|
| `ping` | The extension checks the application is installed and connected. | none |
| `download` | The browser was about to download a file and the extension took it over. | exactly 1 |
| `links` | *Download with EON* on a link or a selection, *Download all links*. | 1–5000 |
| `media` | The person pressed the *Download video* button for a stream the page loaded. | exactly 1, with `media` |
| `refresh` | The person gives an existing download a new address. | exactly 1, with `replaces` |

`start` says what the person expects: `ask` (default) opens the add sheet, `now` starts
at once (silent mode), `later` adds the download stopped.

## What the extension must and must not send

- **`cookies` is the `Cookie` header the browser would send to `url`, and only that.**
  An extension must not send cookies of other sites, nor local storage, nor tokens.
- Only `http` and `https` addresses. The schema rejects anything else, including `file:`,
  `data:`, `blob:` and `javascript:`.
- No DRM-protected streams. `media.type` is `hls` (an `.m3u8` playlist the page itself
  requested) or `file` (a media file the page requested). No site-specific extraction.

## What the application does with it

The application treats every field as untrusted input: names are cleaned before they
touch the file system (no path separators, no reserved Windows names, no leading dots),
sizes are hints until the server confirms them, and cookies are sent only to the host of
`url` (and of `final_url` after a redirect within the same site). Cookies are kept in the
resume state only until the download completes.
