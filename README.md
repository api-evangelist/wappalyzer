# Wappalyzer

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
