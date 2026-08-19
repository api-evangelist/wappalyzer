---
name: Look up the technology stack of a website
description: >-
  Detect the technologies running on one or more websites with the Wappalyzer Lookup API,
  choosing correctly between cached and live results, handling the asynchronous crawl path,
  and staying inside the credit budget.
api: openapi/_original/wappalyzer-v2-public-openapi.yaml
operations: [getCreditBalance, lookupWebsites]
generated: '2026-08-14'
method: generated
source: openapi/_original/wappalyzer-v2-public-openapi.yaml + conventions/wappalyzer-conventions.yml
---

# Look up the technology stack of a website

Base URL `https://api.wappalyzer.com/v2`. Every request needs the header `x-api-key: <your api key>`.
There is no OAuth on the REST API and no scopes — one account key does everything.

## 1. Check the budget first

Call `getCreditBalance` — `GET /credits/balance` — before a batch. It returns `{"credits": <integer>}`
and costs nothing you need to reason about. Do this when you are about to spend more than a handful
of credits, because credit exhaustion returns **403**, the same status as a bad API key, and you
will not be able to tell the two apart from the response alone.

## 2. Decide cached vs live before you call

`lookupWebsites` — `GET /lookup` — defaults to **cached** data up to two months old. That is the
right default for prospecting and the wrong one for "what is this site running right now".

| Intent | Parameters | Credits per URL |
|---|---|---|
| Prospecting, enrichment, bulk | defaults (`live=false`) | 1 |
| Current single-page state | `live=true&recursive=false` | 1 |
| Full site profile | `live=true&recursive=true` | 5 |

Other parameters that change the answer: `denoise` (default `true`, drops low-confidence
detections), `min_age` / `max_age` in months, and `squash` (default `true`, merges monthly results).

## 3. Batch up to ten URLs

`urls` is a comma-separated list and accepts **between one and ten** URLs. More than ten is a
**400**. Ask for extra columns with `sets` — allowed values are `locale`, `email`, `phone`,
`contact`, `social`, `meta`, `security`, `trackers`, `company`, `keywords`, `signals`,
`createdAt`, `events`, `all`. Use `signals` when you want `technologySpend` and `trafficLevel`.

```
GET /lookup?urls=https://example.com,https://example.org&sets=company,signals
x-api-key: <your api key>
```

## 4. A 200 does not mean every URL succeeded

The response is an **array**, one item per URL, and each item is independently one of three shapes
(`LookupResponseItem` is a `oneOf` with no discriminator property). Branch on which fields are
present:

- `technologies` present → completed. Read `technologies[].slug`, `.name`, `.versions`, `.cpe`,
  `.categories[]`, `.confirmedAt`.
- `crawl: true` → **pending**. The site had no cached record and a crawl was started. Any
  `technologies` present are partial.
- `errors` present → that URL failed. `errors` is an array of English strings; there is no error
  code to branch on. Treat the other array items as still valid.

## 5. Handle the asynchronous crawl

`live=true` with `recursive=true` and no cached record completes asynchronously and can take up to
**15 minutes**. Two ways to collect the result:

- **Callback (preferred).** Pass `callback_url=<your https endpoint>`. Wappalyzer POSTs the final
  `LookupCompleted` payload there. Verify it: compute `sha256(signing_secret + rawRequestBody)` and
  compare against the `wappalyzer-signature` header. Signing is **opt-in** and the header is
  declared optional, so decide explicitly whether you accept unsigned callbacks.
- **Polling.** Re-issue the same lookup up to three times, five minutes apart. Do not poll tighter
  than that; the rate limit returns **429** with no `Retry-After` header to guide you.

## 6. Read the meter on every response

Every successful response carries `wappalyzer-credits-spent` and `wappalyzer-credits-remaining`.
These are the only runtime budget signal — there are no `RateLimit-*` headers. Stop the batch when
`wappalyzer-credits-remaining` approaches zero rather than discovering it as a 403.

## Errors

| Status | Meaning | Do |
|---|---|---|
| 400 | Invalid request | Fix it. Retrying unchanged fails again. |
| 403 | Bad key **or** invalid resource **or** out of credits | Call `getCreditBalance` to disambiguate. |
| 429 | Rate limited | Back off. No `Retry-After` is published. |
| 5xx | Transient | Retry with backoff. |

Errors are a bespoke `{"url": ..., "errors": [...]}` envelope, not RFC 9457 problem+json.
See `errors/wappalyzer-problem-types.yml`.
