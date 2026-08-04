# Wappalyzer

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Technology detection REST API for identifying software, frameworks, CMS platforms, analytics tools, and other technologies used on any website. Provides programmatic access to technographic data via lookup, analyze, crawl, and dataset endpoints using a credit-based model.

**Website:** https://www.wappalyzer.com/  
**API Docs:** https://www.wappalyzer.com/docs/api/  
**Pricing:** https://www.wappalyzer.com/pricing/  
**Status:** https://status.wappalyzer.com/  
**GitHub:** https://github.com/wappalyzer  
**LinkedIn:** https://www.linkedin.com/company/wappalyzer  
**X:** https://twitter.com/Wappalyzer  

## APIs

- **Lookup API** — GET `https://api.wappalyzer.com/v2/lookup/` — detect technologies on 1-10 URLs per request, synchronous or async with callback
- **Analyze API** — single-URL technology analysis
- **Crawl API** — deep multi-page recursive crawl with async delivery
- **Dataset API** — bulk pre-built technographic dataset access

## Authentication

All API requests require an `x-api-key` header. Create your key in the API key tab of your Wappalyzer account (eligible plan required).

## Plans

| Plan | Price | API Credits |
|------|-------|-------------|
| Free | $0/mo | 50 |
| Pro | $250/mo | 5,000 |
| Business | $450/mo | 20,000 |
| Enterprise | $850+/mo | 200,000+ |

Annual billing saves 17%. Credits included in plans expire after 60 days; bundle credits expire after 365 days.

## Rate Limits

- 10 requests per second per API key
- Up to 10 URLs per Lookup API request
- 1 credit per URL (cached) / 5 credits per URL (live recursive crawl)
- 30 second request timeout; async crawls up to 15 minutes

---

*APIs.json profile maintained by [Kin Lane](mailto:kin@apievangelist.com)*
