# Zotero-in-Codex

This note documents a practical way to connect Codex to a local Zotero library on Windows and use it for citation verification. LLMs are not accurate with citations and this method can save your time to verify the citations based your Zotero Library.

## What worked
### 1. Confirm the Zotero connector is live
With Zotero open locally, the connector responded on:

`http://127.0.0.1:23119`

The route that worked reliably for context was:

`POST /connector/getSelectedCollection`

That returned the active library and collection.

## What did not work cleanly
The local connector did not provide straightforward item-level access through guessed REST routes. Several attempts to use:

- `/api/users/0/collections/.../items`
- unauthenticated GET requests
- guessed connector search routes

returned `403`, `404`, or `400`.

The lesson is simple: do not assume that the connector exposes a full local REST API for item retrieval just because the connector port is live.

## Reliable fallback
### 2. Read Zotero's active data directory
The active Zotero data directory was stored in:

`C:\Users\XXX\AppData\Roaming\Zotero\Zotero\Profiles\sh1y1zip.default\prefs.js`

The key preference was:

`extensions.zotero.dataDir = C:\Users\XXX\Zotero`

That directory contained the live Zotero database:

`C:\Users\XXX\Zotero\zotero.sqlite`

### 3. Do not query the live database directly
When Zotero is open, `zotero.sqlite` may be locked. The safer workflow is:

1. copy `zotero.sqlite`
2. place the copy in a project temp folder
3. query the copy, not the live file

In this project, the working copy was placed in:

`temp folder\zotero.sqlite`

## Why this worked better
The copied database gave direct access to:
- collections
- collection items
- item titles
- creators
- dates
- DOIs

That made it possible to match manuscript references against a specific Zotero collection by:
- first author
- year
- title
- DOI

## Practical matching rule
Do not match on DOI alone.

Use this order:
1. DOI match when present
2. first author + year + exact title
3. first author + year + close title match
4. manual review for standards and reports

This matters because some non-journal items in Zotero may have:
- abbreviated corporate authors
- shortened titles
- no DOI

## Recommendation
For Codex-assisted manuscript work, use Zotero in two layers:
### Layer 1. Connector
- confirm Zotero is running
- detect the selected collection
- verify context quickly

### Layer 2. Local database copy
- query `zotero.sqlite` through a copied file
- verify item-level metadata
- resolve DOI gaps
- cross-check manuscript references against the actual collection

Use it for:
- author
- year
- title
- DOI
- collection membership

Do not use it as a substitute for reading the paper when the issue is:
- conceptual interpretation
- empirical claim support
- whether a citation truly warrants a sentence in the manuscript
