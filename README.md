# BizBuySell Scraper — businesses for sale and franchises through the Apify API

Get BizBuySell listings into your own code as structured JSON: asking price, cash flow,
revenue, EBITDA, price multiples, the broker's or seller's contact details, the address, every
photo and the full seller write-up, for each business for sale on
[BizBuySell](https://www.bizbuysell.com), plus franchise costs and growth from its franchise
directory. The scraper runs on [Apify](https://apify.com) over plain HTTP, so there's nothing
to host and no browser to run.

**▶ Run it on Apify: [BizBuySell Scraper](https://apify.com/alkausari_mujahid/bizbuysell-scraper)**

This repository holds documentation and examples for developers. The scraper itself runs
on the Apify platform.

---

## Search BizBuySell with one HTTP request

This returns Florida restaurants with at least $100,000 of cash flow in the same response:

```bash
curl -X POST \
  "https://api.apify.com/v2/acts/alkausari_mujahid~bizbuysell-scraper/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "states": ["FL"],
    "categories": ["restaurants-and-food-businesses"],
    "minCashFlow": 100000,
    "maxResults": 20
  }'
```

Your API token is under **Settings → API & Integrations** in [Apify Console](https://console.apify.com).
Add `&format=csv` to the URL for a spreadsheet instead of JSON. The synchronous endpoint
waits up to 300 seconds; for larger runs, start the run and fetch the dataset when it
finishes, as below.

You can also pass any BizBuySell page as `startUrls` instead of filters: a state, city,
county or industry page, a search you set up in the browser, single listings, or franchise
pages.

## From Python

```bash
pip install apify-client
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

# A buy box: profitable service businesses in Texas and Florida with seller financing.
run = client.actor("alkausari_mujahid/bizbuysell-scraper").call(run_input={
    "states": ["TX", "FL"],
    "categories": ["service-businesses"],
    "minCashFlow": 250000,
    "maxAskingPrice": 2000000,
    "sellerFinancingOnly": True,
    "sortBy": "newest",
    "maxResults": 500,
})

for deal in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(deal["title"], deal["askingPrice"], deal["cashFlow"], deal["priceToCashFlow"], deal["contactPhone"])
```

## From JavaScript

```bash
npm install apify-client
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

// Every franchise in a category, cheapest to open first.
const run = await client.actor('alkausari_mujahid/bizbuysell-scraper').call({
    startUrls: [{ url: 'https://www.bizbuysell.com/franchise-for-sale/low-cost-franchise/' }],
    maxResults: 0,
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
items
    .filter((f) => f.type === 'franchise')
    .sort((a, b) => (a.cashRequired ?? Infinity) - (b.cashRequired ?? Infinity))
    .forEach((f) => console.log(f.name, f.cashRequired, f.royaltyFee, f.totalUnits));
```

---

## Input

Every field is optional. With no `startUrls`, the filters define the search; with no
filters either, it searches all of BizBuySell.

| Field | Default | What it does |
|---|---|---|
| `startUrls` | — | BizBuySell URLs: search, state / city / county / industry pages, filtered searches (`?q=`), listing pages, and the franchise directory, categories or brand pages. |
| `keyword` | — | Searches listing titles and descriptions. |
| `states` | — | Two-letter codes for US states, DC and Canadian provinces, e.g. `["FL", "TX", "ON"]`. |
| `categories` | — | Industry slugs, top level or sub-industry, e.g. `"restaurants-and-food-businesses"`, `"pizza-restaurants"`. The full list of 211 is in the Actor's input schema. |
| `listingTypes` | — | Any of `"established"`, `"asset-sale"`, `"real-estate"`, `"startup"`. |
| `minAskingPrice`, `maxAskingPrice` | — | Asking price range in USD. |
| `minCashFlow`, `maxCashFlow` | — | Cash flow (SDE) range in USD. |
| `minRevenue`, `maxRevenue` | — | Gross revenue range in USD. |
| `listedWithinDays`, `updatedWithinDays` | — | Only listings added, or added or changed, in the last N days. |
| `establishedAfter` | — | Only businesses established after this year. |
| `sellerFinancingOnly`, `sbaPrequalifiedOnly`, `realEstateIncludedOnly`, `homeBasedOnly`, `absenteeOwnerOnly`, `priceReducedOnly` | `false` | Keep only listings with that property. |
| `includeRelocatable` | `false` | When filtering by state, also include businesses from elsewhere that can relocate there. |
| `sortBy` | `"relevance"` | `"newest"`, `"price-low"`, `"price-high"`, `"cash-flow-high"` or `"ebitda-high"`. With a limit, you get the top results in this order. |
| `maxResults` | `100` | Records to return across everything. `0` means no limit. |
| `monitorMode` | `false` | Return only listings that are new or changed since the previous run (see below). |
| `monitorName` | derived | Names the monitoring history; by default it's derived from the URLs and filters. |
| `includeRawData` | `false` | Add BizBuySell's original record (170+ fields) as `raw`. |
| `proxyConfiguration` | Apify Proxy | The default datacenter proxy works; no residential proxy is needed. |

## Output

One record per business listing (`"type": "listing"`). This real listing has the contact
person's name and phone replaced, and long text and photo lists shortened:

```json
{
    "type": "listing",
    "id": 2569692,
    "url": "https://www.bizbuysell.com/business-opportunity/profitable-burger-franchise-for-sale-orlando-fl/2569692/",
    "title": "Profitable Burger Franchise for Sale – Orlando, FL",
    "tagline": "Established Turnkey Restaurant | $1.2M Annualized Revenue | 13% Profit",
    "listingType": "Established Business",
    "industry": "American Restaurants",
    "industryGroup": "Restaurants & Food",
    "secondaryIndustry": "Quick Service Restaurants",
    "askingPrice": 620000,
    "cashFlow": 226000,
    "grossRevenue": 1200000,
    "ebitda": 156000,
    "netIncome": 156000,
    "ownerCompensation": 60000,
    "priceToCashFlow": 2.74,
    "priceToEbitda": 3.97,
    "priceToRevenue": 0.52,
    "inventoryValue": 25000,
    "inventoryIncluded": false,
    "ffeValue": 120000,
    "realEstateValue": null,
    "realEstateIncluded": false,
    "realEstate": "Leased",
    "buildingSqFt": 3000,
    "rent": 9800,
    "rentPeriod": "per month",
    "yearEstablished": 2019,
    "employees": 8,
    "employeesFullTime": 4,
    "employeesPartTime": 4,
    "homeBased": false,
    "relocatable": false,
    "absenteeOwner": false,
    "sellerFinancing": false,
    "sbaPrequalified": true,
    "isFranchise": true,
    "location": "Ocoee, FL",
    "streetAddress": "336 Moore Rd",
    "city": "Ocoee",
    "county": "Orange County",
    "state": "FL",
    "stateName": "Florida",
    "zip": "34761",
    "country": "US",
    "latitude": null,
    "longitude": null,
    "description": "Profitable Burger Franchise – $1.2M Annualized Revenue – Orlando Area\n\nAsking Price: $620,000\n\nEstablished, Profitable Restaurant – Turnkey Franchise Opportunity…",
    "facilities": "The Brave Burger is located in an approximately 2,497-square-foot space in Ocoee, Florida.…",
    "competition": "The Brave Burger operates in the competitive Ocoee restaurant market…",
    "growthAndExpansion": "A new owner may have opportunities to grow sales through local marketing…",
    "supportAndTraining": "Training and transition support will be provided…",
    "reasonForSelling": "The franchisor currently owns and operates this location.",
    "website": "http://www.thebraveburger.com",
    "sellerType": "Franchisor",
    "contactName": "Jane Doe",
    "contactFirstName": "Jane",
    "contactLastName": "Doe",
    "contactPhone": "(407) 555-0100",
    "contactTrackingPhone": null,
    "brokerCompany": null,
    "brokerProfileUrl": null,
    "mainImage": "https://images.bizbuysell.com/shared/listings/256/2569692/47a4188a-ec93-4af7-972f-d5ede3a1a890.webp",
    "images": [
        "https://images.bizbuysell.com/shared/listings/256/2569692/47a4188a-ec93-4af7-972f-d5ede3a1a890.webp",
        "https://images.bizbuysell.com/shared/listings/256/2569692/1259ca27-26df-4699-ab75-92ef2757988b.webp"
    ],
    "imageCount": 19,
    "documents": [],
    "adLevel": "Diamond",
    "hotProperty": false,
    "recentlyAdded": true,
    "recentlyUpdated": false,
    "priceReduced": false,
    "views": 50,
    "lastUpdated": "2026-10-28T12:57:20Z",
    "rank": 1,
    "searchUrl": "https://www.bizbuysell.com/florida/restaurants-and-food-businesses-for-sale/",
    "scrapedAt": "2026-10-29T10:33:09.413Z"
}
```

The full record also carries `employeeNotes`, `financingDetails`, `leaseExpiration`,
`leaseTerms`, `franchiseResale`, `franchiseBrand`, `locationDescription`, `contactPhotoUrl`,
`brokerLogoUrl`, `brokerDesignations`, `auctionStart` and `auctionEnd`. Listings posted by a
broker fill `brokerCompany`, `brokerProfileUrl`, `brokerLogoUrl` and `contactPhotoUrl`, and
`documents` lists files such as PDF teasers as `{ "name", "url" }`.

### Franchise records

Franchise pages return `"type": "franchise"` records, here shortened:

```json
{
    "type": "franchise",
    "id": 948,
    "url": "https://www.bizbuysell.com/franchise-for-sale/wayback-burgers/",
    "name": "Wayback Burgers",
    "categories": ["Restaurant and Food", "Multi-Unit", "SBA Approved", "Veteran's Franchises"],
    "industry": "Restaurants (Sit-Down)",
    "website": "https://waybackburgers.com/franchising/",
    "cashRequired": 250000,
    "netWorthRequired": 500000,
    "totalInvestmentMin": 550000,
    "totalInvestmentMax": 650000,
    "averageInvestment": 557500,
    "franchiseFee": 35000,
    "royaltyFee": "$400 + 6.0% of Gross Sales",
    "royaltyPercent": 6,
    "adFundFee": "2.0% of Gross Sales",
    "adFundPercent": 2,
    "successRate": 95.436,
    "continuityRate": 95.221,
    "franchisedUnits": 152,
    "companyOwnedUnits": 1,
    "totalUnits": 153,
    "newUnitsOpened": 9,
    "unitHistory": [
        { "year": 2021, "franchisedUnits": 126, "companyOwnedUnits": 3 },
        { "year": 2025, "franchisedUnits": 152, "companyOwnedUnits": 1 }
    ],
    "franchisingSince": 2002,
    "headquarters": "Cheshire, CT",
    "trainingProvided": true,
    "highlights": ["Training and Support", "Financing Available", "Multi Units"],
    "availableInStates": ["AL", "AK", "AZ", "…"],
    "latestFdd": "2026-02-12T00:00:00",
    "logo": "https://images.bizbuysell.com/fnw/images/963/1021/7dd5bbf9-b2f3-402c-b279-d6f5a0cd931f.jpg"
}
```

They also carry `summary`, `description`, `tagline`, `franchiseFeeText`, `averageUnitRevenue`,
`ceoName`, `homeBased`, `availableInCanada` and `images`.

---

## Things to design around

**One dataset can hold two record types.** Branch on `type`: `"listing"` or `"franchise"`.

**Deduplicate on `type` + `id`.** `id` is BizBuySell's listing number or franchise ID, stable
across runs, so it also lets you match today's prices against yesterday's. Within a run, a
listing found by several searches is returned once.

**Undisclosed numbers are `null`, never `0`.** Sellers often leave cash flow, revenue or EBITDA
blank. `priceToCashFlow`, `priceToEbitda` and `priceToRevenue` are filled only when both
numbers exist.

**Confidential details are `null` too.** If a seller hides the street address, city or
website on BizBuySell, those fields stay empty, and so do the coordinates.

**There's no "date listed".** BizBuySell doesn't publish one. `lastUpdated` is its
last-modified timestamp; for fresh listings use `listedWithinDays`, or `firstSeenAt` in
monitor mode.

**Searches past 10,000 results are split for you.** BizBuySell caps each search at 10,000
results. With a higher `maxResults` (or `0`), an oversized search is split by industry, then
by state, then by sub-industry. A slice split by state covers the US and Canada only.

**Filters combine with pasted search URLs.** Filters narrow a pasted search page, but
`categories` adds to the page's own industry rather than replacing it. Filters don't apply to
listing URLs or franchise pages.

**`?q=` URLs from the browser:** the listing-type filter in them is always applied. If a URL
carries another filter BizBuySell won't translate, the run log names it; pass it as an input
field instead.

**`maxResults` is shared** across all URLs and filters in a run. `rank` is the position
within the search a record came from; `searchUrl` is the pasted URL it came from, or `null`
for a filter search. Listings passed as direct URLs have `rank: null`, and ones that were
taken down are reported in the log.

**Text is plain text.** BizBuySell's markdown is stripped from descriptions, and paragraphs
are separated by blank lines.

## Monitor new and changed listings

With `monitorMode: true`, the first run returns everything that matches, and every later run
returns only listings that are new or whose price, financials or details changed. Each one
adds `changeType` (`"NEW"` or `"UPDATED"`), `firstSeenAt`, and `previousPrice` when the asking
price moved. Unchanged listings are neither returned nor charged, and later runs only look at
listings BizBuySell updated since the last complete run, so they're fast. The history lives in
a named key-value store on your Apify account; use a different `monitorName` for each feed.

```json
{ "keyword": "HVAC", "listingTypes": ["established"], "monitorMode": true, "monitorName": "hvac-deals", "maxResults": 0 }
```

---

## Pricing

Pay per record returned, with full details included; there's no separate per-listing detail
charge. The current rate is on the Actor's
[Pricing tab](https://apify.com/alkausari_mujahid/bizbuysell-scraper/pricing).

Set **Maximum cost per run** in the run options and the scraper works out how many records
that covers, then stops there.

## Use it from an AI assistant

Apify's MCP server lets Claude, Cursor and other MCP clients run this scraper for you. Add
this server URL to your client:

```text
https://mcp.apify.com?tools=alkausari_mujahid/bizbuysell-scraper
```

Then ask in plain words, for example "find HVAC businesses for sale in Texas with at least
$300,000 of cash flow and seller financing".

## Schedules and integrations

On Apify you can run a search on a schedule, get a webhook when a run finishes, and send
results to Google Sheets, HubSpot, Airtable, Zapier, Make, Slack or your own endpoint. All of
it is set up from the [Actor page](https://apify.com/alkausari_mujahid/bizbuysell-scraper).

---

## Support

Questions, bugs or feature requests: open an issue in the
[Actor's Issues tab](https://apify.com/alkausari_mujahid/bizbuysell-scraper/issues), or
email [alkausarimujahid@gmail.com](mailto:alkausarimujahid@gmail.com).

*This is an independent tool. It is not affiliated with, endorsed by or sponsored by
BizBuySell. "BizBuySell" is a trademark of its owner and is used here only to describe the
data source.*
