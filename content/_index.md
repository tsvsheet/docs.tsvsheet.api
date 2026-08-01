---
title: tsvsheet.api
---

**tsvsheet.api** is the sheet API server for [tsvsheet](https://github.com/tsvsheet/tsvsheet): the `tsvsheet-api` binary serves a directory of `.tsvt` spreadsheets over HTTP so clients edit a sheet by sending small semantic operations instead of the whole file, watch it change live, and read its computed values — all through the same [go-tsvsheet](https://github.com/tsvsheet/go-tsvsheet) engine as every other frontend.

- Source: [tsvsheet/tsvsheet.api](https://github.com/tsvsheet/tsvsheet.api)
- Engine: [tsvsheet/go-tsvsheet](https://github.com/tsvsheet/go-tsvsheet)
- Language: [tsvsheet/tsvsheet](https://github.com/tsvsheet/tsvsheet)

## Start a server

```console
tsvsheet-api --root ./sheets
```

The server binds `127.0.0.1:8787` and refuses any non-loopback address: it carries no TLS and no authentication of its own, so wider exposure is an error rather than a risk. `--max-cells` caps the resources any one sheet may use; `--no-compute` serves the document plane only.

## One URL per document

Every document under `--root` is one resource; the HTTP method and media types select the action.

| Request | Meaning |
| --- | --- |
| `GET /budget.tsvt` | the source document (`application/vnd.tsvsheet.doc+tsv`) |
| `POST /budget.tsvt` | apply an edits batch (`application/vnd.tsvsheet.edits+tsv`) |
| `PUT /budget.tsvt` | replace or create the whole document |
| `DELETE /budget.tsvt` | delete the document |
| `GET /budget.tsvt` with `Accept: text/event-stream` | subscribe to the change feed |
| `GET /budget.tsvt` with `Accept: application/vnd.tsvsheet+tsv` | the whole computed grid (also `text/tab-separated-values`, `text/csv`) |
| `GET /budget.tsvt!B7` | one computed cell; ranges (`!A1:C10`) serve row, column, or range shapes |

Every response carries a strong `ETag` — the SHA-256 of the document's canonical bytes — and a `Tsvsheet-Capabilities` header naming what the deployment serves (`edits, events, compute`). Errors are `application/problem+json`.

## Edit without sending the sheet

An edits batch is a TSV document: one operation per line, addressed in the sheet's own A1 notation. `setCell`, `insertRow`, `deleteRow`, `insertCol`, `deleteCol`, `duplicateRow`, `duplicateCol`, `fill`, and `paste` cover the whole edit surface, and a batch applies atomically — any refused line rejects the batch and the sheet is untouched.

```console
rev=$(curl -sI http://127.0.0.1:8787/budget.tsvt | grep -i etag | cut -d'"' -f2)
printf 'setCell\tB7\t=sum(B1:B6)\n' |
    curl -X POST --data-binary @- \
      -H 'Content-Type: application/vnd.tsvsheet.edits+tsv' \
      -H "If-Match: \"$rev\"" \
      http://127.0.0.1:8787/budget.tsvt
```

Mutations require `If-Match`: a stale revision is `412 Precondition Failed`, so two writers can never silently overwrite each other. A batch may also carry its own `#.base <revision>` line — the same conditional check, readable in the batch itself. `Idempotency-Key` makes retries safe: a replayed batch returns its original result.

## Watch a sheet change

The change feed is Server-Sent Events. Each applied batch arrives as a `changed` event carrying the operations with the revisions they fold between, and — on computing deployments — a `computed` event listing exactly the cells whose values changed. Reconnect with `Last-Event-ID` to replay missed events; when the replay window is gone the server sends `reset` and the client refetches.

```text
id: 42
event: changed
data: #.base	9f2c…
data: #.rev	3ab9…
data: setCell	B7	=sum(B1:B6)

id: 43
event: computed
data: #.rev	3ab9…
data: B7	270
```

## Sheets that read sheets

Computed responses use the tsvsheet vendor media types, which are exactly what the language's `IMPORT*` functions request — so any sheet this server computes is importable by any other sheet with no extra machinery:

```text
=importcell("https://sheets.example/budget.tsvt!B7")
```

A server without the compute plane never serves formulas where computed values were requested — a values `Accept` on a document-plane-only deployment is `406 Not Acceptable`, so an importer can never mistake source text for data.

## Editing from Go, local or remote

The same interface serves both: `document.Port` states the whole document plane — read a document, apply an edits batch, replace or create it, delete it — and two implementations satisfy it. `store` edits files in place with no server anywhere, and `client` speaks to a `tsvsheet-api` over HTTP. A program that can edit a local sheet can edit a remote one by swapping which it holds.

```go
// Embedded: no listener, no HTTP, just files under a confined root.
port, err := store.Open(store.RootDir("./sheets"), tsvsheet.DefaultLimits())

// Remote: the same operations against a running server.
port := client.New("https://sheets.example", nil)

snap, err := port.Get(ctx, "budget.tsvt")
applied, err := port.Apply(ctx, "budget.tsvt", batch, snap.Rev)
```

Every mutation carries the revision it expects, so a second writer is refused rather than silently overwriting the first — whichever implementation is in hand. Refusals are the same errors either way: `document.ErrMissing`, `ErrPrecondition`, `ErrExists`, `ErrSyntax`, and the engine's own for a refused batch. A path that is not a clean relative name inside the namespace reads as missing, so whether a refused name exists is never disclosed.

That interchangeability is not a promise, it is a test: the `conformance` package is one table of behaviours run against every implementation, and a new port is expected to run it rather than be trusted.
