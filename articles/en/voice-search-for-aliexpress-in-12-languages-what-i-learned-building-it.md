# Voice search for AliExpress in 12 languages: what I learned building it

*Originally published at [dev.to](https://dev.to/ohadfarkash/voice-search-for-aliexpress-in-12-languages-what-i-learned-building-it-42i4) · OneFindMe article archive · [onefindme.com](https://onefindme.com/)*

AliExpress has a microphone in its app, and it mostly understands English. That is the whole state of voice shopping on the world's largest marketplace. So when I added voice search to [OneFindMe](https://onefindme.com/en/) — a free search layer for AliExpress that works in 12 languages — I expected the hard part to be speech recognition. It wasn't. Here is what actually mattered.

## Recognition was the easy part

Modern browsers ship speech recognition (the Web Speech API). Chrome and Edge on desktop and Android, Safari on iOS — the coverage is good enough that I did not need an audio pipeline, a model, or a server. The user taps the mic, speaks in Hebrew, Arabic, Spanish or Turkish, and the browser hands back text. Free, fast, and the recognition quality in the major languages is better than I could have built.

What the API does *not* give you is a shopping query. It gives you a sentence.

## The real problem: a spoken sentence is not a search

People don't *speak* keywords. Nobody says "wide running shoes". They say "I need running shoes but my feet are wide, something not too expensive". Typed search already suffers from this; voice makes it worse, because speech is longer, more hedged, and full of filler.

So the voice path reuses the same intent layer as typed search, with one extra step: **strip the conversational shell first.** "I need", "something like", "not too expensive", "for my daughter" — these are signals (price sensitivity, recipient), not query terms. The product noun and its qualifiers survive; the rest becomes filters or gets dropped. Only then does the phrase get resolved to a product concept and translated into the short, noun-first query the marketplace responds to.

## Language is not one switch

Twelve languages means twelve recognizer locales, and the browser needs to be told which one *before* the user speaks. Guessing from the site language works most of the time; letting the user override it matters for the Arabic-speaking user on an English page. And two languages needed special handling: Hebrew numbers ("שלושים שקל") and Arabic dialect words for common products, where the recognizer returns spelling variants the keyword map had never seen.

## The button nobody pressed

The feature shipped with a grey microphone emoji and got almost no use for days. The fix was not technical: a labelled orange button that says "voice search" in the user's language. Usage appeared immediately. If you build voice into anything, label it in words.

## What voice actually changed

Voice users search in longer, more natural sentences than typists — which is exactly the input the intent layer is best at. It turned out that the people most likely to speak to the search box are the ones least likely to know the English keyword. Voice and multilingual search are the same feature seen from two sides.

Try it: [onefindme.com](https://onefindme.com/en/) — the orange "voice search" button, any of 12 languages, no app, no signup.
