---
name: Browse the technology and category directory without an API key
description: >-
  Resolve technology and category slugs, read technology profiles, and explore adoption data
  using Wappalyzer's four anonymous metadata endpoints — no API key, no credits.
api: openapi/wappalyzer-metadata-api-openapi.yml
operations: [listTechnologies, getTechnology, listCategories, getCategory]
generated: '2026-08-14'
method: generated
source: openapi/wappalyzer-metadata-api-openapi.yml + mcp/wappalyzer-mcp.yml
---

# Browse the technology and category directory without an API key

Four endpoints on `https://api.wappalyzer.com/v2` require **no authentication and consume no
credits**. They are not in the `/docs/api/v2/` reference; they are the endpoints behind the
`wappalyzer://technologies` and `wappalyzer://categories` MCP resources, and each was confirmed
anonymously reachable (HTTP 200) on 2026-08-14.

Use them for every slug resolution, taxonomy question, and technology-profile lookup, so that
paid credits are spent only on the questions that actually need website data.

## The catalogue

`listCategories` — `GET /categories/` — returns all 106 categories:

```json
{"id": 74, "slug": "a-b-testing", "technologiesCount": 57, "name": "A/B Testing",
 "groups": [{"name": "Analytics", "slug": "analytics"}]}
```

`listTechnologies` — `GET /technologies/` — returns the full directory, 7,281 entries, ~1.3 MB.
Fetch it once and cache it; it is a large payload and the data changes slowly.

```json
{"slug": "11sight", "name": "11Sight", "icon": "11Sight.svg",
 "categories": [{"id": 52, "slug": "live-chat", "name": "Live chat", "priority": 9}],
 "versioned": false, "hostnames": 38}
```

`versioned` tells you whether Wappalyzer can detect a version for that technology at all — check
it before promising version data. `hostnames` is the adoption count.

## Detail

`getCategory` — `GET /categories/{slug}/` — the category's `groups` plus a `technologies` map
keyed by slug, each with `hits` and `hostnames`. This is the cheapest way to answer "what are the
ecommerce platforms and how big is each".

`getTechnology` — `GET /technologies/{slug}/` — the full profile:

- Identity: `name`, `icon`, `website`, `description`
- Commercial: `pricing` (tags such as `low`, `recurring`), `saas`, `oss`
- Adoption: `hostnames`, `hits`, `createdAt` (epoch seconds)
- Distribution: `topIpCountries`, `topLanguages`, `topIndustries`, `topCompanySizes` — maps of
  key to count
- `topHostnames` — the highest-traffic sites running it, each `{hits, www}`
- `alternatives[]` — competing technologies in the same category, as `{slug, categorySlug,
  categoryName}`

## Where this fits

1. Resolving a name a user typed ("Shopify") to the slug the paid APIs require (`shopify`) —
   always do this here first.
2. Building the `technologies[].slug` and `categories[]` filters for a lead list, before spending
   anything. See `wappalyzer-build-lead-list.md`.
3. Answering market-share and competitive questions outright — `hostnames`, `hits`,
   `topIpCountries` and `alternatives` often answer the question without a lookup at all.

## Caveats

- Slugs are lowercase and hyphenated. A miss returns 404, not an empty object.
- These endpoints are undocumented in the public API reference. They are first-party and stable
  enough that Wappalyzer's own MCP server depends on them, but they carry no published
  versioning or deprecation commitment — see `lifecycle/wappalyzer-lifecycle.yml`.
- Only paths under `/v2/` work. `https://api.wappalyzer.com/technologies/` returns
  `403 {"message":"Missing Authentication Token"}`, which is the API Gateway catch-all, not an
  auth requirement.
