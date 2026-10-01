# Shopping search translation is mostly query expansion: what 941 real queries showed

*Originally published at [dev.to](https://dev.to/ohadfarkash/shopping-search-translation-is-mostly-query-expansion-what-941-real-queries-showed-31d9) · OneFindMe article archive · [onefindme.com](https://onefindme.com/)*

I run [OneFindMe](https://onefindme.com/en/?utm_source=devto&utm_medium=article), a search front-end for AliExpress that takes a query in Hebrew, Arabic, Russian, German and eight other languages and turns it into the short English query the marketplace actually indexes. I always described that step as "translation". After exporting 941 of those query pairs and looking at them properly, I don't think it is.

## The data

The pairs come from the engine's keyword map: hand-curated mappings, mappings the engine learned and kept, and a sample of real searches. Every row is a shopper's query, its language, and the English query that retrieved the right products. It is heavily Hebrew (862 of 941 rows), so treat the other languages as examples, not statistics.

It is public under CC BY 4.0: [Kaggle](https://www.kaggle.com/datasets/onefindme/multilingual-shopping-queries), [Hugging Face](https://huggingface.co/datasets/fufu1976/multilingual-shopping-queries) and [Zenodo (DOI 10.5281/zenodo.22930843)](https://doi.org/10.5281/zenodo.22930843). The full analysis is a [Kaggle notebook](https://www.kaggle.com/code/onefindme/what-shoppers-type-vs-what-marketplaces-index); every number below comes from it.

## Finding 1: the query usually gets longer

If this were translation, you would expect the English side to be about as long as the source, or shorter once the filler is gone. For Hebrew:

| | share of queries |
|---|---|
| English query has **more** words | 59.5% |
| same number of words | 30.4% |
| **fewer** words | 10.1% |

Average length goes from 2.35 words to 3.10. The rewrite adds information far more often than it removes it.

## Finding 2: what gets added is who it is for, and what form it comes in

The most frequent words on the English side are not product nouns. They are **audience** words and **bundle** words:

- 37% of English queries contain an audience word (women, men, baby, kids…)
- 15% contain a bundle word (set, kit, pack, pcs)

Some curated pairs (glosses in brackets):

| shopper typed | meaning | marketplace query |
|---|---|---|
| חצאית | skirt | `women skirt` |
| טיולון לתינוק | stroller for a baby | `baby lightweight stroller` |
| שייקר | shaker | `protein shaker bottle mixer` |
| מגהץ | iron | `garment steamer handheld clothes` |
| דיפ פאודר | dip powder | `dip powder nail kit` |
| נעלי טרום הליכה | pre-walking shoes | `baby prewalker shoes soft sole` |

The Hebrew for "skirt" doesn't need the word "women"; context and grammatical gender carry it. A listing title on AliExpress does need it, because that is how sellers write titles and how the marketplace's search matches them. The same goes for "shaker": in Hebrew, in a shopping context, it means the gym bottle. The English word alone is ambiguous.

## Finding 3: when it gets shorter, it is dropping noise

The 10% that shrink are mostly shoppers putting *conditions* into the query:

| shopper typed | meaning | marketplace query |
|---|---|---|
| כפכפי נשים מותג משלוח חינם | women's slippers brand free shipping | `women slippers brand` |
| מצעים תקציב נמוך | bedding, low budget | `bedding set` |

"Free shipping" and "low budget" are filters, not product words, and putting them in the search string hurts more than it helps.

## What I changed my mind about

- **It is expansion, not translation.** A plain MT model gives a correct "skirt" and drops the word that listing titles lead with. The target isn't the most faithful English sentence. It is the query in the vocabulary the listings are written in.
- **Evaluate on retrieval, not fluency.** BLEU against this target column would reward the right words, but the real test is whether the query returns the intended products. A fluent but literal translation can score fine on text overlap and still retrieve the wrong things.
- **Curate the head, generate the tail.** The curated rows are the high-traffic queries, checked by hand. The long tail is rewritten by a language model with instructions to produce marketplace vocabulary, and the results are cached so the next shopper with the same query gets an instant answer.

## Caveats

The data is mostly Hebrew. Rows marked `observed` are tagged with the language of the page the search was made on, not the language of the text, so a few carry a tag that doesn't match their script. I haven't measured retrieval quality per row here, only the shape of the rewrites.

If you build multilingual search for e-commerce, I'd be curious whether you see the same expansion pattern in your languages.

*Written with AI assistance; the numbers are computed from the dataset in the linked notebook.*
