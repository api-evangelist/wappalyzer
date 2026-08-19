---
name: Build and finalize a technographic lead list
description: >-
  Run the full two-phase Wappalyzer lead-list flow — define the technology query, wait for the
  asynchronous calculation, price and inspect the sample, then commit credits to finalize and
  download the rows.
api: openapi/_original/wappalyzer-v2-public-openapi.yaml
operations: [getCreditBalance, listCategories, listTechnologies, createLeadList, getLeadList, listLeadLists, finalizeLeadList, deleteLeadList]
generated: '2026-08-14'
method: generated
source: openapi/_original/wappalyzer-v2-public-openapi.yaml + data-model/wappalyzer-data-model.yml
---

# Build and finalize a technographic lead list

A lead list answers the inverse question — not "what does this site use" but "which companies use
this technology". It is the only Wappalyzer resource with a lifecycle, and it is a **two-phase
commit**: calculating a list is free, finalizing it spends credits.

Base URL `https://api.wappalyzer.com/v2`, header `x-api-key: <your api key>`.

## 0. Resolve the technology slugs first — for free

`createLeadList` filters on technology **slugs** and category **slugs**, and a wrong slug produces
an empty or wrong list you may have already paid for. Resolve them against the anonymous metadata
endpoints, which need no API key and spend no credits:

- `listCategories` — `GET /categories/` — 106 categories.
- `listTechnologies` — `GET /technologies/` — the full directory (7,281 entries as of 2026-08-14).
- `getCategory` — `GET /categories/{slug}/` — the technologies inside one category.

## 1. Create the list

`createLeadList` — `POST /lists`. The body is a query, not a record. Useful fields:

- `technologies[]` — each `{slug, operator, version}`; `operator` is `=`, `>=` or `<=`.
- `categories[]`, `keywords[]`, `languages[]`, `countries[]`, `industries[]`, `companySizes[]`,
  `tlds[]`.
- `matchTechnologies` — `or` | `and` | `not`. This one silently changes the meaning of the whole
  list; set it deliberately.
- `matchCountryLanguage`, `excludeNoTraffic`, `excludeMultilingual`.
- `minAge` (0-11) / `maxAge` (1-12) months, `fromDate` (epoch seconds).
- `subset` and `subsetSlice` (0-4) to take a slice of a large result.
- `requiredSets[]` / `sets[]` — the columns in the export. `signals` adds `technologySpend` and
  `trafficLevel`.
- `callbackUrl` — where to receive the ready notification.

The response is `CreateListAccepted`: `{"id": "lst_...", "status": "Calculating"}`. **Save the id.**

> There is no idempotency key on this endpoint. If the request times out, do **not** blindly retry —
> call `listLeadLists` first and check whether a list was already created, or you will create a
> duplicate.

## 2. Wait for it to calculate

Status moves through `ListStatus`: `Calculating` → `Ready` | `Failed` | `Insufficient`.

- **Callback.** If you passed `callbackUrl`, Wappalyzer POSTs a `ListReadyCallback`:
  `{id, status: "Ready", rows, setRows, totalCredits, sampleUrl}`. Verify with
  `sha256(signing_secret + rawRequestBody)` against the `wappalyzer-signature` header.
- **Polling.** `getLeadList` — `GET /lists/{id}` — returns the `ListDetail` with the current
  `status`.

`Insufficient` means too few rows matched — loosen the query, do not retry it unchanged.

## 3. Price it before you commit

At `Ready` you have `totalCredits` (what finalizing will cost), `rows` (how many companies), and
`sampleUrl` (a downloadable sample of the rows). This is the free preview. Do three things here:

1. Fetch `sampleUrl` and confirm the rows are the companies you meant.
2. Compare `totalCredits` against `getCreditBalance` — `GET /credits/balance`.
3. Only then decide to spend.

If the list is wrong, `deleteLeadList` — `DELETE /lists/{id}` — and rebuild. Deleting before
finalizing costs nothing.

## 4. Finalize and download

`finalizeLeadList` — `POST /lists/{id}` with body `{"spendCredits": <integer>}`. `spendCredits` is
required and is your explicit authorization to spend; pass the `totalCredits` you verified in
step 3. The response is `FinalizeListResponse`: `{"id", "status": "Complete", "url"}` — `url` is
the download.

> This is the credit-spending call and it has no replay protection. A retry after a timeout can
> double-spend. If a finalize call does not return cleanly, call `getLeadList` and check whether
> `status` is already `Complete` before trying again.

## 5. Housekeeping

`listLeadLists` — `GET /lists` — returns every list as `ListSummary` (`id`, `createdAt`, `status`,
`totalCredits`, `rows`, `technologies`, `keywords`). Use it to reconcile after any ambiguous
create or finalize.

## Errors

`400` invalid query, `403` bad key or insufficient credits, `429` rate limited. `403` is
overloaded — call `getCreditBalance` to tell a credit problem from an auth problem. See
`errors/wappalyzer-problem-types.yml`.
