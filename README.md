# Amazon Scraper API: what ScraperAPI costs per 1,000 Amazon requests, can you get full reviews and BSR, and which plan fits your volume? (full plan comparison table)

ScraperAPI bills an Amazon request at **5 credits**; an ordinary web page costs 1. That single multiplier decides your budget. Startup, the tier most small teams land on, carries 1,000,000 monthly credits, which is 200,000 Amazon page fetches for **$149/month**, or **$134.10/month** on annual billing ($1,609.20 for the year). The free plan's 1,000 credits are 200 Amazon requests, not 1,000.

Two cases where ScraperAPI is the wrong buy: you need the full review history for tens of thousands of ASINs (no product-page scraper returns it, see below), or you want the largest free sandbox for a one-off data quality test (Bright Data hands out 5,000 free records a month). Everything else here is about matching a plan to the number of requests you actually run.

> **Quick pick.** Wire up your pipeline on the free plan first: 1,000 credits, 5 concurrent threads, no card. Move to **Startup at $149/month ($134.10 annual)** when it goes to production, since 1,000,000 credits covers 200,000 Amazon requests with 50 concurrent threads. Hobby's 100,000 credits (20,000 Amazon requests, 20 threads) is enough for one category refreshed weekly. Business and above exist for country-level geotargeting and unlimited analytics history.
> 👉 [Compare all ScraperAPI plans and current prices](https://www.scraperapi.com/pricing?fp_ref=coupons)

## All ScraperAPI plans, with the Amazon request count each one buys

Every paid tier shares the same credit pool across general pages, Amazon, Google SERPs and social sites, so the "Amazon requests" column is simply credits ÷ 5. Nothing else in the plan changes the exchange rate.

| Plan | Credits / month | Amazon requests / month | Concurrent threads | Price | Sign up |
| --- | --- | --- | --- | --- | --- |
| Free | 1,000 | 200 | 5 | $0 | [Start free](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | 100,000 | 20,000 | 20 | On the plan card | [See Hobby pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 200,000 | 50 | **$149/mo · $134.10/mo annual** | [See Startup pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 600,000 | 100 | On the plan card | [See Business pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 1,000,000 | 200 | On the plan card | [See Scaling pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 2,100,000 | 300 | On the plan card | [See Professional pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 4,300,000 | 500 | On the plan card | [See Advanced pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 4,400,000+ | 500+ | Quoted | [See Enterprise terms](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Three differences live outside the credit count. Geotargeting is US and EU only on Hobby and Startup, and country-level from Business upward. Analytics history is 30 days on those two entry tiers and unlimited from Business. Annual billing takes roughly 10% off, using Startup's $149 versus $134.10 as the reference.

## What one Amazon request actually returns

The documented route for Amazon product data is a structured endpoint that takes an ASIN and a marketplace and hands back JSON or CSV, with JSON as the default. Query parameters are `api_key`, `asin`, `country_code` and `tld`. If you'd rather not hold a connection open, the async version accepts a single `asin` or an `asins` array plus a webhook callback, so you can post a batch and collect it later.

The `tld` field covers 22 Amazon marketplaces: com, co.uk, ca, de, es, fr, ie, it, co.jp, co.za, in, cn, com.sg, com.mx, ae, com.br, nl, com.au, com.tr, sa, se and pl. `country_code` sets where the request appears to come from, and pairing it with `tld` is how you see, say, amazon.com shipping information as a Canadian shopper would.

The payload is a product record plus a specification block. A trimmed example from the documentation:

json
{
  "name": "CUCKOO Twin Pressure Rice Cooker 6-Cup ...",
  "product_information": {
    "brand": "CUCKOO",
    "capacity": "3 Quarts",
    "product_dimensions": "13.5\"D x 9.5\"W x 9.25\"H",
    "item_weight": "12.76 pounds",
    "voltage": "120 Volts"
  }
}


Variant details (size, colour, other options) come back with the product where the listing has them. For the deeper end of what an Amazon record can hold, Apify's product actor documents the field list: `bestsellerRanks` with category and rank, `coupon`, `discountPercent`, `starsBreakdown`, `monthlyPurchaseVolume`, `isAmazonChoice`, `isBestSeller`, `variantAsins`, seller and `shipsFrom`, return policy, plus an `aiReviewsSummary`. That list is a useful checklist for what to demand from any vendor, including this one.

## How the credits burn, and what that means per 1,000 requests

Amazon's 5-credit base is only the starting number. Adding `render=true` adds 10 credits, and `premium=true` adds another 10, so a JavaScript-rendered Amazon page costs 15 and your 1,000,000 credits buy about 66,600 of them rather than 200,000. Ultra premium sits at 30 credits and 75 with rendering, and it's restricted to paid plans. Geotargeting, sessions, device type, output format and auto parsing add nothing.

Two controls keep the bill predictable. Set `max_cost` on a request to cap what a single scrape may spend, and read the `sa-credit-cost` response header to see what the call actually cost. The dashboard's Domain Cost Estimator prices any URL before you send it.

Now the comparison that matters, because the three main vendors publish pricing in three different units:

- **ScraperAPI** bills per request in credits. At Startup, $149 ÷ 200,000 requests works out to about **$0.75 per 1,000 Amazon requests**.
- **Oxylabs** bills per delivered result, with Amazon requests **from $0.50 per 1,000 results**. On unit price at the entry paid tier, its published rate undercuts ScraperAPI's Startup math for Amazon-only work.
- **Bright Data** bills per successful record and gives new accounts **5,000 free credits every month**, which it describes as up to 5,000 Amazon records. It doesn't publish a per-record figure in its docs, so get the rate from its pricing page before you model anything.
- **Apify** bills per result through individual actors, and the Amazon actor's unit price isn't shown on its listing page.

The per-request versus per-result distinction changes who carries the risk. ScraperAPI charges its 5 credits when the request is made; Oxylabs and Bright Data charge for results delivered. If your targets fail often, per-result billing is the friendlier structure.

A worked example. 100,000 ASINs refreshed once a month is 500,000 credits, which fits Startup with half the allowance spare. Refresh weekly and you're at about 2.2 million credits, so Business. Refresh daily and it's 15 million credits, which lands on Advanced at 21.5 million.

👉 [Check which plan covers your credit burn](https://www.scraperapi.com/pricing?fp_ref=coupons)

## The review question: what "all publicly available reviews" really means

ScraperAPI's product endpoint states that it returns all publicly available reviews for an ASIN. Read the qualifier literally. Since late 2024 Amazon requires a login to page through a full review list, so what a product page exposes is roughly the top 8 to 13 reviews plus the "Customers say" summary. Apify documents exactly this constraint for its own Amazon actor: top reviews from the product page, plus Amazon's review summary, not every review.

That ceiling applies to every vendor that reads the product page, so "can your API get reviews" is the wrong question. The right one is whether you need 12 reviews per ASIN or 12,000. For sentiment sampling across a category, a dozen recent reviews per product is usually enough and the product endpoint covers it. For review-text mining, you need a vendor that sells reviews as a separate dataset, and Bright Data is the one in this comparison that lists a dedicated Reviews dataset with reviewer details, verified purchase status and review dates.

Bestseller rank sits in a similar place. It's visible on the page and some vendors surface it as a named field (Apify's `bestsellerRanks`), but category layouts differ. If BSR is a hard requirement, spend 5 of your 20 free Amazon requests testing one ASIN per category before you pay for a year.

## ScraperAPI vs Bright Data, Oxylabs and Apify

| Vendor | Pricing model | Amazon coverage | Request model | Documented limits worth knowing |
| --- | --- | --- | --- | --- |
| ScraperAPI | Credits per request; Amazon = 5 credits | Structured product endpoint by ASIN, 22 marketplaces; any Amazon URL through the general endpoint | Sync by default; async structured endpoint with webhook and ASIN arrays | US/EU geotargeting on Hobby and Startup only; pay-as-you-go starts at Scaling |
| Bright Data | Pay per successful record; 5,000 free credits/month | Separate datasets for products, reviews, sellers, global products and search | Sync `/scrape` for one input, 1-minute ceiling; async `/trigger` up to 1 GB per job, unlimited URLs | Real-time only, no published response time; production traffic should use async or webhooks |
| Oxylabs | From $0.50 per 1,000 results, delivered results only | Endpoints for any URL, bestsellers, offers by ASIN, product by ASIN, keyword search, sellers, plus a Q&A source | Sync, async or batch per endpoint | Seller parsing is desktop-only; `run_js` needs a client timeout of 150s or more; cloud storage delivery is async-only |
| Apify | Actor-based, per result | One row per product with price, BSR, coupons, star breakdown, variant matrix, top page reviews | Sync API returns within 5 minutes; longer jobs run and poll | Top 8–13 reviews only; no ZIP targeting; no other-sellers panel; about 20 search pages maximum; brand store URLs unsupported |

**When each one beats ScraperAPI.** Bright Data wins if reviews or seller profiles need to be first-class datasets rather than fields, or if you want records delivered straight into Snowflake, S3 or GCS. Oxylabs wins if Amazon is the entire job and you want a published per-result rate below ScraperAPI's Startup equivalent, with postal-code level location targeting. Apify wins if you want the richest single-product record, BSR and AI review summaries included, without maintaining a parser, at volumes small enough that per-result pricing doesn't bite.

## Where ScraperAPI gets awkward

Geotargeting is the practical constraint. Hobby and Startup give you US and EU only, and country-level targeting starts at Business. If your price monitor compares the same ASIN across a dozen marketplaces with local delivery estimates, budget for Business rather than discovering the limit after launch.

Overage behaviour needs a decision up front. On Hobby, Startup, Business, Scaling and Professional, running out of credits pauses scraping until renewal unless you switch on auto-upgrade, which bumps you to the next plan automatically. Pay-as-you-go, the option that keeps requests flowing at a fixed per-credit rate under a monthly cap, only appears from Scaling upward. A price monitor that spikes during Prime Day will stop mid-spike on Startup.

The platform-level limits are shared with everyone in this category. Amazon shows about 20 pages per search (split by category or price band for more), brand store URLs aren't supported by page-level scrapers, and sponsored placements need filtering so they don't pollute your result set. None of these are reasons to pick a different vendor; they're reasons to design around the source.

👉 [Start on the free plan with 1,000 credits](https://www.scraperapi.com/pricing?fp_ref=coupons)

## FAQ

**How many Amazon requests do I get on the free plan?** 200. The free plan includes 1,000 credits with 5 concurrent connections, and Amazon bills 5 credits per request. That's enough to validate fields, auth and wiring, not enough to build a price sample.

**Do I handle proxies and CAPTCHAs myself?** No. Proxy rotation, CAPTCHA and anti-bot handling, automatic retries, session support and user-agent rotation are included in the per-request price, along with a 99.9% uptime guarantee and unlimited bandwidth.

**Per request or per result?** ScraperAPI charges per request, Oxylabs and Bright Data charge per delivered result or successful record, Apify per result through each actor. If your failure rate is high, the per-result vendors absorb more of it.

**Can I pull the price for a single ASIN?** Yes. The structured product endpoint takes one `asin` plus `tld`, returns JSON by default, and the async variant accepts an `asins` array with a webhook for batches.

**Which Amazon sites are covered?** 22 `tld` values, from com and co.uk through co.jp, com.au, com.br, com.sa and com.tr. Each marketplace carries its own ASINs, so the ASIN from amazon.com won't resolve on amazon.de without the matching local identifier.

## Picking a plan without overspending

Match the tier to request volume, not to feature envy. A handful of ASINs checked occasionally runs on the free plan indefinitely. One category refreshed weekly, up to 20,000 Amazon requests a month, fits Hobby. A production feed around 100,000 ASINs refreshed monthly, or Amazon data mixed with Google SERP monitoring, lands on Startup at $149/month. Daily full-catalogue refreshes need Business or above, and that's also the point where country-level geotargeting and unlimited analytics history arrive.

Two things to confirm before you enter card details: how your specific request mix bills (base 5 credits, plus 10 for rendering, plus 10 for premium proxies), and what happens when you hit the cap on your chosen tier, since auto-upgrade is opt-in and pay-as-you-go doesn't exist below Scaling.

👉 [See the Startup plan at $149/month, or $134.10 billed annually](https://www.scraperapi.com/pricing?fp_ref=coupons)
