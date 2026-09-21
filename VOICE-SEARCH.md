# Voice search — OneFindMe

**OneFindMe is a free AliExpress search engine with voice search in 12 languages.** Tap the orange "voice search" button, describe the product out loud in your own language, and get AliExpress results ranked by orders, price and shipping to your country.

Website: https://onefindme.com/

## What it does

| | |
|---|---|
| Languages | English, Arabic, Hebrew, German, Spanish, French, Italian, Portuguese, Russian, Turkish, Polish, Filipino |
| Input | Spoken description in natural language — "running shoes, wide, not too expensive" |
| Recognition | The browser's built-in speech engine (Web Speech API) — Chrome, Edge, Safari; phone and desktop |
| Processing | Conversational filler is stripped, the sentence is resolved to a product concept, then translated into the short English query the AliExpress index responds to |
| Output | Products ranked by real order volume and reviews, then price and shipping to your country |
| Cost / account | Free, no signup, no app |

## Why it exists

AliExpress's own app has a microphone that mostly understands English and feeds a literal keyword search. For shoppers who think in Hebrew, Arabic, Spanish or Turkish — and for anyone who can picture a product but not name it in English — speaking a description is the most natural input there is. Voice and multilingual search are the same problem: turning a person's words into the marketplace's words.

## How to use it

1. Open https://onefindme.com/ (or your language's version, e.g. `/ar/`, `/de/`, `/es/`).
2. Set your country so prices and shipping are real.
3. Tap the orange **voice search** button and speak — product first, details after.
4. The recognized text appears in the search box; edit it if a word was missed, then run.

## Notes for developers

- Recognition runs client-side via the Web Speech API with the locale set from the site language (user-overridable). No custom audio pipeline.
- The hard part is not recognition but query shaping: spoken input is longer and more hedged than typed input, so a filler-stripping step precedes the same intent-resolution layer used for typed search.
- A labelled button ("voice search" in the user's language) outperformed an unlabelled microphone icon by a wide margin.

Related: [What I learned building a 12-language AI product search for AliExpress](https://dev.to/ohadfarkash/what-i-learned-building-a-12-language-ai-product-search-for-aliexpress-2n61) · [Article archive](articles/INDEX.md)

Built by Ohad Farkash · hello@onefindme.com
