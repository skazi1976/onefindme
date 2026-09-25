# Adding eBay to an AliExpress search engine: what the numbers said

*Originally published at [fufu2004.mataroa.blog](https://fufu2004.mataroa.blog/blog/adding-ebay-to-an-aliexpress-search-engine-what-the-numbers-said/) · OneFindMe article archive · [onefindme.com](https://onefindme.com/)*

I run [OneFindMe](https://onefindme.com/en/), a free search front-end for AliExpress: you describe a product in your own language (typed, spoken or as a photo) and a language model turns that into the short English keywords AliExpress actually matches on. This week I added eBay as a second source. Before building anything I ran the same searches against both stores and looked at the numbers, because "more stores" sounds obviously better and I wasn't sure it was.

## The test

One intent, "wireless earbuds", new items only, fixed price, asked from two countries. eBay's Browse API lets you say where the shopper is (a `contextualLocation` header), so prices and delivery estimates come back for that country, not for the US.

**Germany.** eBay returned 40 listings and all 40 shipped from inside Germany. Most had free shipping and an estimated delivery of 1–3 days. The median price was €14.75. The AliExpress results for the same query had a median of €27.29, and delivery measured in weeks.

**Greece.** A completely different picture. eBay's listings shipped from Germany, the UK, the US and China, with €8–23 shipping on top, no delivery estimate, and a median of €58.58. AliExpress's median was €14.21, with items that had 10,000–30,000 orders behind them.

The lesson was immediate: whether a second marketplace helps depends almost entirely on **where the shopper is**. In a country where eBay has a big local seller base, it wins on the thing AliExpress is worst at, which is delivery time. In a small market it is a worse deal on every axis.

## What that meant for the design

- **eBay is a separate row, never mixed into the ranking.** The AliExpress grid stays exactly as it was; eBay appears underneath as "Also on eBay". Interleaving the two would have meant inventing a common relevance score for two catalogues that describe products very differently.
- **Only items that ship to you.** Every query filters on the visitor's delivery country. Countries with their own eBay site (US, UK, Germany, France, Italy, Spain, Australia, Canada and a few more) search it directly; everyone else searches eBay US and only sees what actually ships to them.
- **A shipping sanity rule.** In Brazil a dog bed came back at R$377 with R$789 of shipping. It was technically a valid result, but nobody buys that, so any item whose shipping costs more than twice its price is dropped.
- **"Fast delivery" is claimed only when it is true.** The source filter above the results (All / AliExpress / eBay) shows a fast-delivery hint on the eBay option only if at least three of that search's eBay items arrive within a week in that country. From Israel eBay takes weeks too, so the hint never appears there.
- **The default is always "All".** eBay's affiliate commission is 1–4% by category; AliExpress's is higher on most of what people search. Showing the cheaper-to-me store first would be the honest-looking way to be dishonest, so the visitor picks, and nothing is pre-selected.

## The bug the test exposed

The German comparison surfaced something unrelated and more embarrassing. The query "kabellose Kopfhörer" (wireless headphones) returned a hedge trimmer, a floor cleaner and a baby monitor. The engine only translated queries written in non-Latin scripts; German, being Latin script, went to AliExpress raw, and AliExpress matched "kabellose" (cordless) against everything cordless.

The fix was to translate every language except English, regardless of script. Measured on a fixed set of German queries, the share of relevant results went from 74% to 95%. It is a good reminder that "the language uses our alphabet" and "the marketplace understands it" are different claims.

## Why Amazon isn't in the results (yet)

The obvious third source is Amazon, and people asked for it. Amazon only lets affiliates display its products (images, prices) through its Creators API, which replaced the old Product Advertising API in May 2026, and access requires 10 qualifying sales through your account in the previous 30 days, per marketplace. Scraping the data instead is against the Associates agreement.

So the connection is built but dormant: every search checks whether the visitor's Amazon marketplace qualifies (the refusal is cached for a few hours so nobody waits on it), and until it does, the source bar offers a plain "Amazon" link that runs the same search on the visitor's local Amazon store. The day a marketplace crosses the threshold, its product row appears on its own.

## Takeaways

1. Measure per country before adding a source. The same store can be the best and the worst option depending on where the shopper stands.
2. Keep sources side by side rather than merged, unless you can defend a single ranking across them.
3. Only claim a benefit (speed, free shipping) when the data for that exact search supports it.
4. Test in the languages you support, not just the one you think in. The worst bug here had nothing to do with eBay.

*Written with AI assistance; the numbers are from the engine's own test runs and logs.*
