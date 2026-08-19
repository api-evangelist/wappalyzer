---
name: Profile a domain's subdomains and verify contact emails
description: >-
  Enumerate a domain's website-serving subdomains, profile the stack on each, and verify
  associated email addresses — the reconnaissance and enrichment flow across the Subdomains,
  Lookup and Verify APIs.
api: openapi/_original/wappalyzer-v2-public-openapi.yaml
operations: [getCreditBalance, lookupSubdomains, lookupWebsites, verifyEmail]
generated: '2026-08-14'
method: generated
source: openapi/_original/wappalyzer-v2-public-openapi.yaml + conventions/wappalyzer-conventions.yml
---

# Profile a domain's subdomains and verify contact emails

Base URL `https://api.wappalyzer.com/v2`, header `x-api-key: <your api key>`.

Wappalyzer infers technologies from **public website signals only**. This is external
reconnaissance and enrichment, not vulnerability scanning, and it does not expose hidden internal
software. Keep that boundary in any output you generate.

## 1. Enumerate the subdomains

`lookupSubdomains` — `GET /subdomains`. One domain per request.

```
GET /subdomains?domains=example.com&limit=100
x-api-key: <your api key>
```

Parameters:

- `limit` — default 100, **between 10 and 1000, and a multiple of 10**. Anything else is a 400.
- `after` — the pagination cursor.

The response is `SubdomainsResult`: `{domain, subdomains, moreAfter}`. Note the shape —
`subdomains` is an **object keyed by hostname**, not an array, and each value is a
`SubdomainRecord` of `{createdAt, updatedAt}` epoch seconds. The hostname lives in the key.

Paginate by passing the returned `moreAfter` value back as `after`. Stop when `moreAfter` is
absent.

## 2. Profile the interesting hosts

Feed the hostnames into `lookupWebsites` — `GET /lookup` — in batches of **up to ten** URLs:

```
GET /lookup?urls=https://app.example.com,https://shop.example.com&sets=meta,company
```

Budget first. A cached lookup is 1 credit per URL; `live=true&recursive=true` is 5. A domain with
200 subdomains is 200 credits at the cheapest rate, so filter before you spend — `updatedAt` on
each `SubdomainRecord` tells you which hosts Wappalyzer has seen recently.

Remember the response is an array of mixed `LookupCompleted` / `LookupPending` / `LookupError`
items. See the technology-lookup skill for the branching rules.

## 3. Verify contact addresses

`verifyEmail` — `GET /verify` — one email per request.

```
GET /verify?emails=someone@example.com
```

The response is `VerifyResult`, and twelve of its thirteen fields are required, so the shape is
stable and you can read it without null-checking:

- `reachable` — the verdict: `safe` | `risky` | `invalid` | `unknown`. Branch on this.
- The evidence behind it: `syntaxValid`, `mxValid`, `connection`, `deliverable`, `disabled`,
  `inboxFull`, `catchAll`, `disposable`, `roleAccount`.

Practical reading: `catchAll: true` means the domain accepts everything, so `deliverable` proves
nothing — that is what usually produces `reachable: "risky"`. `roleAccount: true` (info@, sales@)
is deliverable but is not a person. `disposable: true` is a throwaway address.

## 4. Meter and errors

Read `wappalyzer-credits-spent` / `wappalyzer-credits-remaining` on every response and stop before
you hit zero — exhaustion returns **403**, indistinguishable from a bad key. Call
`getCreditBalance` — `GET /credits/balance` — to disambiguate.

`400` invalid request (check the `limit` multiple-of-ten rule first), `429` rate limited with no
`Retry-After`. See `errors/wappalyzer-problem-types.yml`.
