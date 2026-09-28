---
name: origin-api
description: >-
  Routes questions about the Cursor Origin API to the right section of the
  Origin docs and names the few rules to check first. Use when a task mentions
  Origin, the Origin API, Origin Apps, or Origin webhooks, including creating
  an Origin App, authenticating as one, calling Origin endpoints, or handling
  Origin webhook deliveries.
license: MIT
compatibility: >-
  Needs network access to https://cursor.com/docs/api/origin/* at run time.
---

# Origin API

The docs are the source of truth. Do not name an endpoint, scope, event slug,
header, or limit from memory. Where this file and the docs disagree, the docs
win.

The docs live under `https://cursor.com/docs/api/origin/`. `llms.txt` is
the index and links every section, endpoint, and webhook payload.
`openapi.yaml` is the contract, and "Endpoint reference" explains its
`x-origin-*` extensions. `llms-full.txt` is the whole reference in one file.
`changelog` says what moved. For one question, read `llms.txt` and fetch
only the section that answers it. Fetch `llms-full.txt` or `openapi.yaml`
whole when you need all of it, as a porting brief does. Cite `operationId`s
and section names.

## Where to look

| Question | Section |
| --- | --- |
| Which credential for which call; minting and lifetime | "Authentication" and its subsections |
| Install flow and the callback receipt | "Installation", "Installation receipt" |
| Which scope an operation needs | `x-origin-scopes` on the operation; "Scopes" |
| What an installation can do on a mirrored repository | "Mirrored repositories" |
| Webhook headers, signature, delivery format, retries, pausing, recovery | "Webhooks" |
| Which events exist and which arrive without subscribing | "Events" |
| Payload shapes | "Event payloads" |
| Pagination, errors, request IDs, repository paths, IDs | "Common conventions" |
| Rate limits | "Rate limits" |
| Check-run keys, attempts, stale writes | "Check runs" |
| What is not there yet | "Current limitations" |
| A checklist to build against | "Implementation checklist" |

## Rules to check first

1. **Native or mirror.** An installation keeps its full scopes only on
   native repositories (created on Origin) and stable outbound mirrors
   (Origin is the source and pushes to GitHub). Some writes work only on
   native repositories, such as merging a pull request or changing the
   default branch, so read each operation's description for mirror limits.
   On a repository mirrored from GitHub, every event except
   `repository.pushed` still arrives, and every call beyond metadata and
   contents reads returns `403` ("Mirrored repositories", "Events").
2. **Subscribe.** Only `installation.*` events arrive without a subscription.
   A missing subscription produces silence, not an error ("Events").
3. **Verify, dedupe, acknowledge.** Verify the signature over the raw body
   before parsing, dedupe on the delivery ID, return `2xx`, then process
   ("Signature verification", "Retries", "Automatic disable"). Origin signs
   a digest, which Standard Webhooks does not, so a generic verifier fails.
4. **Scopes from the spec.** Request the union of `x-origin-scopes.scopes`
   over the operations the app calls ("Scopes").
5. **Opaque tokens and IDs.** Do not build or parse page tokens or IDs
   ("Pagination", "IDs").

Porting an existing GitHub App: use `port-github-app-to-origin` in this
plugin.
