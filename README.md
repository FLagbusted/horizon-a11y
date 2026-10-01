# horizon-a11y

Accessibility fixes for Shopify's **Horizon** theme (base commit `5acd1b6`, 21 Sep 2026), as two small patches.

I scanned live Shopify stores built on Horizon with [a11y-snapshot](https://github.com/FLagbusted/a11y-snapshot)
(axe-core, WCAG 2.2 A/AA). Several failures came from the theme's own markup rather than from the
store's content, so every store on Horizon inherits them. This patch fixes four of them in the theme code.

Shopify is not accepting contributions to Horizon, so this is published as a patch that store owners
and developers can apply to their own copy. Not affiliated with Shopify.

## What it fixes

| # | Issue (axe rule) | WCAG | Where in Horizon | Fix |
|---|---|---|---|---|
| 1 | Hero overlay link has no accessible name (`link-name`) | 2.4.4, 4.1.2 | `sections/hero.liquid` | `aria-label` from a new optional **Link label for screen readers** setting, falling back to the theme's existing "View more details" string |
| 2 | Star rating uses `aria-label` on a plain `<div>`, which most screen readers ignore (`aria-prohibited-attr`) | 4.1.2 | `blocks/review.liquid` | add `role="img"` so the label is exposed |
| 3 | Slideshow thumbnail and dot buttons use `aria-selected`, which is not allowed on `<button>` (`aria-allowed-attr`) | 4.1.2 | `snippets/slideshow-controls.liquid`, `snippets/product-media-gallery-content.liquid`, `assets/slideshow.js`, `assets/zoom-dialog.js` | switch to `aria-current="true"` in markup, JS and the CSS selectors that style the active item |
| 4 | Product media scroller cannot be reached with the keyboard (`scrollable-region-focusable`) | 2.1.1 | `snippets/slideshow.liquid`, `snippets/product-media-gallery-content.liquid` | new optional `scroller_label` parameter; when set, the scroller gets `tabindex="0"`, `role="region"` and a label ("Slideshow: {product title}"). Used for the product media gallery only |
| 5 | Recommendation loading placeholders put `aria-label` on plain `<div>`s (`aria-prohibited-attr`) | 4.1.2 | `sections/product-recommendations.liquid`, `blocks/product-recommendations.liquid` | one `role="status"` container with a visually hidden "Loading product recommendations" message; placeholders get `aria-hidden="true"` (patch 0002) |

Patch 0001: 7 files changed, 43 insertions, 15 deletions. Patch 0002: product-recommendation loading placeholders now expose one `role="status"` message instead of a prohibited `aria-label` on each placeholder `<div>` (2 files, +6 −3).

## How it was checked

- **Shopify Theme Check:** the same 6 pre-existing offenses on stock and patched Horizon. The patch adds none.
- **JavaScript:** both changed files pass `node --check`.
- **axe-core on the affected patterns:** `verify/before.html` reproduces the markup the scanner flagged on live Horizon stores; `verify/after.html` is the same markup as the patch renders it. Result: 4 failing elements (4 rules) before, 0 after.
- **Live store, 1 Oct 2026:** stock Horizon 4.2.0 vs the same theme with both patches, on a Shopify development store with the same products and templates, scanned with a11y-snapshot (home, collection, product, cart). Reports in `results/live-2026-10-01/`.

  | | Failing elements | Issue types | Critical |
  |---|---:|---:|---:|
  | Stock Horizon 4.2.0 | 26 | 6 | 3 |
  | Horizon + 0001 + 0002 | 17 | 3 | 0 |

  Fixed on the live store: slide-control `aria-allowed-attr` (3, all critical), recommendation placeholders `aria-prohibited-attr` (4), hero `link-name` (1), product media scroller `scrollable-region-focusable` (1). An earlier run with 0001 alone gave 21 / 4 / 0.
  The review-rating fix could not be exercised there because the test product has no reviews block; it is covered by `verify/`.
- **What is left, and why:** 12 `scrollable-region-focusable` on product-card galleries (see Known issues); 4 `frame-title` on Shopify's preview-bar iframe, which exists only in theme preview, not on a live storefront; 1 `list` on the shadow-DOM `overflow-list`, whose `<ul>` contains `<slot>` elements that receive `<li>` children (likely a tool limitation, not yet confirmed with a screen reader).

## Known issues not fixed here

- **Product-card image galleries** (`snippets/card-gallery.liquid`) are scrollers inside a link. Making them focusable would nest interactive elements, so this needs a design change (for example, arrows that are always visible, or one image per card), not a one-line fix.
- **Video poster images** with no alt text come from the output of Shopify's `video_tag` filter, not from theme markup.
- **Colour contrast** failures depend on each store's colour-scheme settings, not the theme code.

## Apply

    cd your-horizon-theme
    git apply /path/to/0001-*.patch /path/to/0002-*.patch
    shopify theme dev            # preview, then re-scan before publishing

Then, in the theme editor, fill in **Link label for screen readers** on any Hero section that has a link.

## License

The patch modifies Horizon, which is © Shopify Inc. under the Horizon license (copy in `HORIZON-LICENSE.md`),
which allows derivative works for Shopify themes. My own files (README, verify pages) are MIT.
