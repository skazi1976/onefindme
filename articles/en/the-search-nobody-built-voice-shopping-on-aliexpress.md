# The Search Nobody Built: Voice Shopping on AliExpress in Twelve Languages

*Originally published at [medium.com](https://medium.com/@ohadf1976/the-search-nobody-built-voice-shopping-on-aliexpress-in-twelve-languages-f01cb4baec6a) · OneFindMe article archive · [onefindme.com](https://onefindme.com/)*

Ask ChatGPT, Claude or Gemini for "a voice search engine for AliExpress" and they agree: there isn't one. The AliExpress app has a microphone, it mostly understands English, and whatever it hears goes into the same literal keyword matcher that fails typed queries too.

I found that odd, because voice is the most natural input for exactly the shopper AliExpress serves worst: the one who can picture a product perfectly and cannot name it in English. "The little stone for face massage." "The thing that holds a baby's pacifier." "Running shoes, but wide." The catalog is indexed in English; a word-for-word translation of any of those returns noise.

I run [OneFindMe](https://onefindme.com/en/), a free search layer for AliExpress that takes natural-language descriptions in 12 languages and turns them into the marketplace's own query. Adding voice to it taught me three things, in reverse order of what I expected.

**Recognition is a solved problem — for free.** Modern browsers ship speech recognition. Chrome, Edge and Safari handle the major languages well; I did not build a model or an audio pipeline. Tap, speak, get text.

**A spoken sentence is not a search.** People don't speak keywords. They hedge, they add context, they say "something like" and "not too expensive." Typed search already struggles with this; voice makes it worse. The real work was a step *before* search: strip the conversational shell, keep the product noun and its qualifiers, turn "not too expensive" into a price filter instead of a query term — then resolve the phrase to a product concept and translate it into the short, noun-first query the index responds to.

**Label the button.** The feature launched as a grey microphone icon and was ignored for days. Replacing it with an orange button that says "voice search" in the user's language changed everything overnight. Icons are for people who already know what they're looking for; words are for everyone else.

The unexpected part: voice users search in longer, more natural sentences than typists — precisely the input a meaning-first engine handles best. Voice search and multilingual search turned out to be one feature seen from two sides.

If you want to try it: onefindme.com, the orange button, any of twelve languages, no app, no account. And if you know another tool that does voice search on AliExpress in more than English, tell me — I looked, and I couldn't find one.

*Originally published in longer, more technical form on [Dev.to](https://dev.to/ohadfarkash/voice-search-for-aliexpress-in-12-languages-what-i-learned-building-it-42i4).*
