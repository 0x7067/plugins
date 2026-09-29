---
name: port-github-app-to-origin
description: >-
  Plans the port of an existing GitHub App to a Cursor Origin App. Use when the
  task is to bring a GitHub App to Origin or compare what it uses against the
  Origin API. Reads the app's needs out of its code, maps them onto the live
  Origin spec, and writes a porting brief with feedback for Cursor. Planning
  only.
license: MIT
compatibility: >-
  Needs network access to https://cursor.com/docs/api/origin/* at run time.
---

# Port a GitHub App to an Origin App

Run this inside the app's codebase. The output is a porting brief for the
team plus a Feedback for Cursor section they can send as is
(`references/brief.md`). This skill plans. It does not write or change code
unless the user asks for that after reading the brief. Follow the
`origin-api` skill for the docs and the rules to check first. Two more rules:

1. **Find it in the code, do not ask.** Read what the app uses out of its
   source. Anything you cannot find becomes a question for the team.
2. **Feedback describes use cases, not the team's code.** The team's parts of
   the brief may cite `file:line`. Feedback for Cursor says only what the app
   needs to do and what Origin lacks for it, in Origin terms, with no file
   paths, module names, framework details, or repository names.

## What to find

Record `file:line` for each item, and note what you looked for but did not
find.

- Declared permissions and events, from a manifest or infrastructure code
  if one is checked in. Otherwise derive them from the calls.
- Every webhook event the app handles, and every payload field each handler
  reads, including fields it only logs.
- Every REST and GraphQL call, with the parameters and filters the code
  passes, the response fields it reads, whether it runs on every webhook or
  in a loop, and how it paginates.
- Authentication: the JWT algorithm, how the app learns the installation ID
  after install, how it handles token expiry, any user sign-in and what it is
  for, and whether the app clones or pushes git.
- The webhook receiver: how it verifies signatures, whether it has the raw
  body when it verifies, and how it deduplicates deliveries.
- Calls a framework or library makes for the app. Probot's receiver, token
  cache, and config loader; Octokit `App`'s installation and repository
  listing; app-auth libraries. Read the dependency's docs and list these
  calls marked "from `<dependency>`".

## How to map

A brief needs the whole spec, but the files are too large to read into
context (about 870 KB and 575 KB). Download them to disk and search them:

```bash
curl -sSL https://cursor.com/docs/api/origin/openapi.yaml -o openapi.yaml
curl -sSL https://cursor.com/docs/api/origin/llms-full.txt -o llms-full.txt
rg -B1 -A4 'x-origin-scopes:' openapi.yaml        # every operation with its scopes
rg -A3 'x-origin-webhook-events:' openapi.yaml    # every event slug with its payload schema
rg -n '^### Get Pull Request$' llms-full.txt      # then read to the next ### heading
```

Read only the sections a match points at. Do not open `openapi.yaml` or
`llms-full.txt` with a plain file read; file readers stop after about 50 KB,
so find the line with `rg -n` and read that line range. Each endpoint and
each webhook payload type has a section in `llms-full.txt` that spells out
its fields as dotted paths. For a single question later, `llms.txt` names the one section
to fetch.

- A matching name is a candidate, not an answer. Read the operation's
  description, parameters, and response fields against what the code passes
  and reads. If a parameter or field the code depends on is missing, that is
  a workaround or a gap, not a match.
- GitHub's pull request calls that live under `/issues/{n}/…` (comments,
  labels) live under the pull request endpoints on Origin. If the code uses
  them on real issues, see "Where GitHub features live on Origin" in
  `references/brief.md`.
- Origin has no GraphQL. Break each query into REST calls and record the
  fan-out.
- The scopes to request are the union of `x-origin-scopes.scopes` over the
  operations you named. Do not translate the GitHub manifest.
- Each GitHub event and action pair maps to at most one slug in "Events". The
  action is part of the slug. A pair with no slug is not an event on Origin.
- For each payload field the code reads, record whether it is in the
  payload, in the delivery envelope around the payload (`event.type` carries
  the action), needs an extra read (say which operation and how many calls
  per event), can be computed from other fields, or is missing. Payloads are
  snapshots. A field the REST resource has but the payload lacks needs an
  extra read.
- A capability the docs never mention is not available today. Ask the team
  about it. A behavior the docs neither confirm nor deny gets a question plus
  a step in the end-to-end test that checks it. Do not assume it works the
  way it did on GitHub.

## Before finishing

Every claim about Origin points at something in the downloaded files. Every gap
has a feedback entry that names its cost. The feedback reveals nothing about
the team's internals. The summary names the native-or-mirror question.
