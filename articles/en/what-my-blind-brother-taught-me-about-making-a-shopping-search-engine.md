# What my blind brother taught me about making a shopping search engine work with VoiceOver

*Originally published at [dev.to](https://dev.to/ohadfarkash/what-my-blind-brother-taught-me-about-making-a-shopping-search-engine-work-with-voiceover-1npe) · OneFindMe article archive · [onefindme.com](https://onefindme.com/)*

My brother is blind. When I asked him to try the shopping search engine I'd been building, [OneFindMe](https://onefindme.com/accessibility/), with VoiceOver on his iPhone, the automated checker had already told me the site was in decent shape. The first thing he tried, voice search, did nothing at all. That sent me looking for everything else the checker couldn't see.

This is what changed, what I got wrong, and the parts that were more interesting than I expected.

## axe is about a third of the job

I started where everyone starts: axe-core against the home page and a results page, desktop and mobile. It found real problems. Two sort/filter `<select>`s had no accessible name, 130 to 250 grey 7px labels failed contrast, and there was no `main` landmark. All fixed, and the scan went to zero.

Then I went through the site the way my brother uses it, keyboard and screen reader only, and a different list appeared, none of it visible to axe:

- **Pop-up windows that didn't exist for a screen reader.** Favourites, price alerts and "similar products" were plain `<div>`s. They opened on screen, focus stayed behind them, and VoiceOver said nothing.
- **Results that arrived in silence.** A sighted user sees the grid fill in. A blind user had no idea the search had finished.
- **The wrong things first.** The "trending" rail sits above the results in the DOM, so a screen reader reading from the top met a row of unrelated bestsellers before the thing you searched for.
- **Dead buttons.** A ☰ menu button, "Home" and "History" in the bottom bar: visible, focusable, and wired to nothing. For a sighted user that's a small annoyance. For a screen reader user it's a trap.
- **Emoji read aloud.** "Fire. Daily deals." "Bell. Price alerts." Every one of them.
- **Dark-mode bugs that hit everyone.** A white "Load more" button with light text on it (1.2:1), and a help card with the same problem. I'd only ever looked at dark mode on the results page.

## One sentence per product

A product card is a link wrapping an image, a badge, two prices, a title, a rating and an order count. Read in DOM order, that came out as "Top pick, minus sixty-six percent, 15.04, 30.70, Baby cup…". I replaced it with one `aria-label` on the link:

```js
const a11yLabel = [
  shownTitle,
  `${sym}${p.price}` + (p.discount > 0 ? ` (${t.was} ${sym}${p.original_price})` : ""),
  p.rating ? `${t.rating} ${p.rating.toFixed(1)} ${t.of5}` : "",
  orders > 0 ? `${orders.toLocaleString()} ${ordersLabel}` : "",
  p.free_shipping ? t.freeShip : ""
].filter(Boolean).join(", ");
```

The Save and Share buttons now say *which* product they belong to. Forty identical "Save" buttons in a row are useless.

## Announcing results

A polite live region, plus one detail that took me a while to find: a screen reader only speaks a live region when its text *changes*. Two identical searches in a row were silent. Clearing the region first and filling it on the next tick fixes that:

```js
function announce(msg) {
  el.textContent = "";
  setTimeout(() => { el.textContent = msg; }, 60);
}
```

After a search I also move focus to the results heading with `focus({ preventScroll: true })`. The reading position jumps to the results, and a sighted visitor sees nothing move. I skip this while focus is in the filter bar, because arrowing through a `<select>` fires `change` on every step.

## "Describe this product"

This is the part I'm proudest of, and the one I most expected to get wrong.

On AliExpress the titles are keyword piles ("2026 Summer Men's Dad Sneakers Breathable…"). The real information is in the photo, which a blind shopper can't see. So every result now has a button, visually hidden but reachable by keyboard and screen reader, that sends the product photo to a vision model. The model describes what is actually shown, in the page's language.

Some notes:

- **Model choice mattered more than I thought.** I compared Haiku, Sonnet and Opus on the same photos in Hebrew. Haiku invented parts that weren't there. For someone who can't check the photo, a confident wrong description is worse than none. Sonnet was as accurate as Opus and about two seconds faster, which matters when someone is waiting.
- **The prompt says what not to do.** Describe only what is visible. Never invent sizes, brands or specifications. If unsure what a part is, leave it out.
- **Cache per product and language.** Each product is paid for once, about 0.6 cents. The second request is instant.
- **Allowlist the image host.** The endpoint only accepts AliExpress image URLs. Otherwise it's a free vision API on my key. The first version rejected *every* real result, because result cards load images through my own `/img?url=` proxy. It now unwraps that.

## Voice search and VoiceOver

Two surprises:

1. **Don't announce "listening".** My first instinct was to have the page say "Listening, speak now". But VoiceOver would read that text out loud while the microphone is open, and the recogniser would happily transcribe it as your search. So the page stays silent while listening, and only speaks when something goes wrong.
2. **iOS fails silently.** When speech recognition isn't allowed (for example with Dictation turned off), Safari's recogniser throws an error and the page just went quiet. Every failure path now says what happened and offers the keyboard's own dictation microphone, which works in every browser.

## Why I built a menu instead of installing an overlay

Accessibility overlays (the widget with the wheelchair icon) are popular, and they don't make a site accessible. Many blind users actively dislike them, because they fight with the screen reader. My brother's VoiceOver run needed nothing extra.

What *is* useful for low-vision visitors is a small, boring menu: three text sizes, high contrast, stop animations, highlight links, a readable font, and a voice-search language picker. I built my own, and it stays out of the screen reader's way.

Two gotchas from that:

- **The site is laid out in px**, so changing `font-size` did nothing. `body.style.zoom` does what browser zoom does: it enlarges and reflows, with no horizontal scroll on a 390px screen.
- **One page monkey-patched `document.getElementById`.** For missing ids it returned a truthy no-op stub. Every "does my element already exist?" check said yes, so the menu, the live region and the stylesheet were never created. That page now gets a `querySelector`-based lookup.

## Where it stands

- axe: zero violations on the home page in Hebrew, English and Arabic (light and dark), and on results pages in Hebrew and English (desktop and mobile).
- Keyboard: skip link, visible focus everywhere, dialogs that trap focus and return it on Escape, background `inert` while a dialog is open.
- Real use: my brother, with VoiceOver, on his phone. His verdict after the fixes: everything is accessible the way it should be. He's still using it and sending notes.

It's in beta, and some product titles still arrive in English because sellers write them that way. That's exactly what the describe button is for.

*Disclosure: OneFindMe is free, with no signup, and funded by affiliate commissions from the stores, at no extra cost to buyers. I wrote this with help from an AI assistant; the code, the measurements and the testing with my brother are real.*
