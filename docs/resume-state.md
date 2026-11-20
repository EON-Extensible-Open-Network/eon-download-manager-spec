<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 EON contributors
-->

# Resume state (v0)

Schema: [`resume.v0.schema.json`](../schemas/resume.v0.schema.json).

While `report.pdf` downloads, two files sit in its folder:

- `report.pdf.part` — the data, pre-sized to the full length when the length is known,
  written at offsets by every connection;
- `report.pdf.eondl` — this document: where every segment has got to.

## Rules

1. **Segments** are `[start, end)` ranges with a cursor `pos`: bytes `start..pos` are on
   disk, `pos..end` are still to fetch. Segments never overlap, and together they cover
   `[0, length)` exactly.
2. **Splitting** a segment `{start, pos, end}` at `m` (with `pos < m < end`) produces
   `{start, pos, m}` and `{m, m, end}`. That is the whole of dynamic segmentation, as far
   as the file is concerned.
3. The state is written **atomically** (write `report.pdf.eondl.tmp`, flush, rename over)
   **at least once a second** while downloading and on every pause or stop. After a crash
   the data file can hold more bytes than the state admits — never fewer — so resuming
   from the state is always safe.
4. **Validators.** `etag` and `last_modified` are the ones the server gave at the start.
   On resume, requests carry `If-Range`; a full `200` instead of `206 Partial Content`
   means the file changed and the download starts again (unless the person chose to ignore
   changes, for servers that change `Last-Modified` on every request).
5. **Not resumable** (`resumable: false`): the server ignores ranges or gave no length;
   there is one connection, `segments` is empty, and a stop restarts from zero. The
   interface says so before the download starts.
6. **HLS.** `hls.done` lists the media segments already written into the
   `report.pdf.eondl.d/` folder; the playlist is fetched again on resume.
7. On completion the data file is truncated to `length`, renamed to its final name (never
   overwriting: `report (1).pdf`), and the state file is deleted — with it any cookies it
   held.
