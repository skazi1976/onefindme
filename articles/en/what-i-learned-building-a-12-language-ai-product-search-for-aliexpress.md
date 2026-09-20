# What I learned building a 12-language AI product search for AliExpress

*Originally published at [dev.to](https://dev.to/ohadfarkash/what-i-learned-building-a-12-language-ai-product-search-for-aliexpress-2n61) · OneFindMe article archive · [onefindme.com](https://onefindme.com/)*

AliExpress has one of the largest product catalogs on the planet, but its search
assumes you already know the exact English keyword. If a shopper can only *describe* what they
want — "that little stone thing for face massage" — the results collapse into noise. I spent a
while building [OneFindMe](https://onefindme.com/en/), a free multilingual search layer that sits in front of AliExpress,
and these were the lessons that actually moved the needle.

## 1. Translating the *intent*, not the words

Naive translation is a trap. "לק ג'ל" (Hebrew) machine-translates to "gel polish", which on
AliExpress surfaces floor lacquer as often as nail products. The fix was to resolve the shopping
*intent* to a canonical product query first, then translate that — not translate the raw phrase.
A small curated keyword map beat the general model for the high-traffic terms, because the model
kept inventing plausible-but-wrong category ids.

## 2. Only the first few words of a query survive

Long, descriptive queries rank worse than short ones on the marketplace API. The product noun has
to lead. "comfortable running shoes for wide feet women" performs far worse than "wide running
shoes". So the pipeline trims to the product noun + one or two qualifiers before it ever hits the
API.

## 3. Rank by orders, not price

The cheapest listing is almost never the best answer. Sorting candidates by real order volume,
then lightly penalising listings with no reviews, produced results people actually clicked. Price
is a filter, not a ranking signal.

## 4. Cache the translation, not the results

Product availability changes hourly, but the translation of "wireless earbuds" into a good query
does not. Caching the *query resolution* (and letting unknown terms cache their translation on
first use, so the second shopper gets an instant answer) cut latency dramatically without serving
stale stock.

## 5. Every market is different

The same query needs different handling per country: shipping thresholds, which categories pay,
even modesty defaults in some regions where the marketplace's own "relevance" surfaces things a
shopper did not ask for. Filtering the junk while never filtering a legitimate intent turned out
to be the hardest, most locale-specific part.

If you want to see the result, [OneFindMe](https://onefindme.com/en/) is live and free — type what you want in any of 12
languages and it does the translating, trimming, and ranking described above. Happy to answer
questions about any of these in the comments.
