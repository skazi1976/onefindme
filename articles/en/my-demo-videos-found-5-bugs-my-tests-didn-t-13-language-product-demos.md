# My demo videos found 5 bugs my tests didn't: 13-language product demos from real screenshots

*Originally published at [dev.to](https://dev.to/ohadfarkash/my-demo-videos-found-5-bugs-my-tests-didnt-13-language-product-demos-from-real-screenshots-1a6k) · OneFindMe article archive · [onefindme.com](https://onefindme.com/)*

I run a small search engine that sits in front of AliExpress: you describe a product in your own language, typed or spoken, and it turns that into the short English keywords the marketplace actually matches on. The site has pages in 12 languages and understands queries in 21.

I wanted a short demo video for each language. Recording 13 screen captures by hand, on a phone, from 13 "countries", was not going to happen, so I generated them from the live site instead. The videos turned out fine. The more useful result was that making them exposed five bugs that my checks had never caught.

{% youtube 2qjGN7GBHq8 %}

## The pipeline

Four steps, all Python:

1. **Capture.** Headless Chrome through Selenium, with mobile emulation at 412 px wide and a device pixel ratio of 2. For each query the script loads the real search URL, waits until the product grid has real cards, and saves one tall screenshot plus the positions of the elements it needs later: the search box, the results heading, the store filter bar and the voice button.
2. **Pretend to be somewhere else.** The site decides the shipping country and currency from Cloudflare's `/cdn-cgi/trace` endpoint, fetched with a synchronous XHR. I didn't want a German video showing Israeli shekels, so before the page loads the script patches `XMLHttpRequest` to answer that one request with `loc=DE`:

```python
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {"source": r"""
  (() => { const O = XMLHttpRequest.prototype.open, S = XMLHttpRequest.prototype.send;
    XMLHttpRequest.prototype.open = function (m, u) { this.__trace = String(u).includes('/cdn-cgi/trace'); return O.apply(this, arguments); };
    XMLHttpRequest.prototype.send = function () {
      if (!this.__trace) return S.apply(this, arguments);
      Object.defineProperty(this, 'status', { value: 200 });
      Object.defineProperty(this, 'readyState', { value: 4 });
      Object.defineProperty(this, 'responseText', { value: 'fl=1\nloc=DE\n' });
    }; })();"""})
```

   Note the `r"""`. My first version was a normal string, Python turned `\n` into a real newline inside a JavaScript string literal, the script died silently, and every "German" capture came back in shekels.
3. **Compose.** PIL draws each frame: a phone frame on the left that scrolls from the typed query down to the results, and the query and a caption on the right.
4. **Encode.** `imageio-ffmpeg` pipes raw frames into libx264. Set `macro_block_size=1`, or it quietly pads 1080 to 1088 and stretches everything.

A second script reuses the same screenshots for vertical 1080×1920 Shorts. It keeps the site address fixed at the top of the frame, because links in Shorts descriptions aren't clickable.

## Right-to-left and CJK without raqm

My PIL build has no raqm, so it can't shape complex scripts on its own. Three things had to be handled by hand:

- **Hebrew:** reorder with `python-bidi`. But **wrap in logical order first, then reorder each line.** Wrapping an already reordered string splits it in the wrong places.
- **Arabic:** run `arabic_reshaper` before bidi, or the letters don't join.
- **Japanese:** there are no spaces, so wrapping has to break per character. The fonts are also different: Inter has no CJK glyphs, and my first Japanese render showed the query as a row of black boxes.

## The five bugs the videos found

This is why I'm writing this up. To make a video I had to *look* at every screen, in every language, as a visitor from that country.

**1. A Hebrew button on every page.** The share button next to the results heading said "שתף" (Hebrew for "share") in German, French and Polish. There were three copies of the label in the code. My grep for the Hebrew word found one. The other two were written as `\u05e9\u05ea\u05e3` escapes, and a text search for Hebrew never sees those.

**2. English in the middle of German.** The related-search chips for German, Italian, Polish and Filipino fell back to English, so users saw suggestions like "kabellose kopfhörer **for men**". The dictionaries for those languages simply didn't exist.

**3. "Baby night lamp for women."** The chip generator added gender suffixes to every query, so a specific search got a nonsense suggestion. It now adds suffixes only to queries of one or two words.

**4. Everyone on the English page was American.** The English page is also the home of every country without its own page. Its settings mapped to US/USD, so a visitor from Tokyo or London got US shipping, dollar prices, and an eBay-US row whose "fast delivery" badge was true only in the US. The fix maps the visitor's real country to a currency. Before enabling it, I tested country-plus-currency pairs live against the search API: every non-euro currency individually, the euro countries by sampling. 36 countries are on the list now. India and Indonesia returned zero products when shipping there, so they stay on the default.

**5. Results I refused to show.** In Turkish the translation was right, but the marketplace's own ranking for Turkey was thin: most listings had about one order, and one smartwatch had a phone number printed on the image. In Arabic, "small cabin bag for a flight" translated correctly to "cabin travel bag" and still returned motorcycle crash bars, which is a marketplace problem, not a translation one. In both cases I cut the scene. A demo that has to hide weak results tells you something too.

## What I took from it

Generating marketing material from the live product works like an end-to-end test with human eyes: every language, every locale, the real page. It found five real bugs in a single afternoon, none of which any of my existing checks had flagged. The rule I'm keeping: capture, look at every frame, and cut anything you wouldn't want a user to see, instead of retouching it.

The engine is at [onefindme.com](https://onefindme.com/en/) if you want to try breaking it in your language.

*Written with AI assistance. The numbers and bugs come from the actual captures and fixes.*
