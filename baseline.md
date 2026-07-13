# Baseline

## Main PageSpeed Insights Scores

### Homepage mobile scores

- **Performance:** 56
- **Accessibility:** 74
- **Best Practices:** 96
- **SEO:** 100

### Homepage mobile field data

- **Core Web Vitals Assessment:** Failed
- **Largest Contentful Paint (LCP):** 3.2 s
- **Interaction to Next Paint (INP):** 100 ms
- **Cumulative Layout Shift (CLS):** 0
- **First Contentful Paint (FCP):** 2.9 s
- **Time to First Byte (TTFB):** 1.0 s

### Notable secondary score for the audit scope

- **La Tunisie toute l'annee:** mobile **Performance 47**
- **Tunisia Live:** Performance 52, Accessibility 75, Best Practices 73, SEO 100

## Mobile Baseline From The Primary Page

The baseline for this project starts with the homepage: https://www.discovertunisia.com/

Everything in this baseline should be read as **mobile-first** unless noted otherwise. The rendering metrics below come from the throttled mobile PageSpeed run.

### Primary page baseline metrics

- **Core Web Vitals Assessment:** Failed
- **Performance:** 56
- **Field data:** LCP 3.2 s, INP 100 ms, CLS 0, FCP 2.9 s, TTFB 1.0 s
- **Lab data:** FCP 5.1 s, LCP 14.5 s, TBT 80 ms, CLS 0, Speed Index 14.9 s

### Rendering and UX observations

- The homepage fails Core Web Vitals on mobile mainly because LCP is too slow.
- The most concerning rendering metrics are LCP 14.5 s in lab data, Speed Index 14.9 s, and FCP 5.1 s.
- INP 100 ms and CLS 0 are comparatively healthy and should be preserved during optimization.

## Mobile Networking Baseline From The Primary Page

These networking stats were measured on the homepage using a **mobile viewport** with **approximate Slow 4G throttling**, after clearing the browser cache once and then reloading the page for a soft refresh.

### Homepage mobile networking stats

- **Cold-load requests:** 111
- **Cold-load transferred:** 1,747,393 bytes, about 1.67 MiB
- **Cold-load total resource size:** 2,942,799 bytes, about 2.81 MiB
- **Cold-load encoded size:** 1,715,293 bytes, about 1.64 MiB
- **Approximate compression reduction:** about 41.7% overall from decoded resource size to encoded size

### Soft refresh and caching

- **Soft-refresh requests:** 189
- **Soft-refresh transferred:** 1,163,742 bytes, about 1.11 MiB
- **Reduction due to caching:** 583,651 bytes less transferred than the cold load, or about 33.4% less network transfer
- **Cached requests on soft refresh:** 164 of 189 requests had zero network transfer in the browser timing data

### Asset mix

- **Images:** 33 requests, 1,270,876 bytes transferred, about 1.21 MiB
- **JavaScript:** 36 requests, 275,397 bytes transferred, about 0.26 MiB
- **CSS:** 37 requests, 184,355 bytes transferred, about 0.18 MiB
- **Other:** 5 requests, 16,765 bytes transferred

### Compression notes

- **Images:** effectively no additional transfer compression in the browser timing data; encoded and decoded sizes were nearly identical, so the main issue is image weight rather than missing gzip or Brotli
- **JavaScript:** encoded 374,878 bytes vs decoded 1,366,396 bytes, so JavaScript is compressed in transit
- **CSS:** encoded 192,315 bytes vs decoded 690,271 bytes, so CSS is also compressed in transit

### Networking observations

- Even under throttled mobile loading, the homepage still transfers about 1.67 MiB on the first view.
- Images are still the dominant problem: about 1.21 MiB and roughly 73% of cold-load transferred bytes.
- Soft-refresh transfer is still over 1 MiB, which means repeat mobile visits are better but still not lightweight.
- CSS and JavaScript are compressed in transit, but there are still too many separate front-end files for a mobile-first homepage.

---

## Bundle Analysis

### JavaScript

**How JavaScript is bundled:**
JavaScript is not bundled at all. The site loads 41 individual JavaScript files, each delivered as a separate HTTP request. This is the default Drupal 7 behaviour with JS aggregation disabled. There is no route splitting, component splitting, or code splitting of any kind.

**Script loading strategy:**

- 39 of 41 script tags are placed in the `<head>` element
- 37 of those 39 first-party scripts carry no `defer` or `async` attribute, making them render-blocking
- Only the Google Analytics and Google Tag Manager scripts use `async`
- No `<link rel="preload">` hints are used for any resource on the page

**Key libraries loaded:**

- jQuery 1.10.2 — released in 2013, heavily outdated
- Bootstrap 3.3.5 JS — outdated; loaded twice: once from `cdn.jsdelivr.net` and again from the theme's own `/sites/all/themes/bootstrap/js/bootstrap.js`
- Three competing slider libraries loaded on every page: bxslider, flexslider, and views_slideshow
- Two competing lightbox libraries loaded on every page: magnific-popup and lightbox2
- KolorTools / KolorBootstrapOntt — a niche virtual-tour library that is only relevant on virtual-tour pages
- Google Maps API — loaded via a synchronous blocking `<script>` tag with no `defer` or `async`

**Source maps:** No source maps are served with any JavaScript file. This is acceptable for production, but it does mean that minified production code cannot be debugged without rebuilding locally.

**Is this the right decision?** No. Loading 41 unbundled synchronous scripts in the `<head>` dramatically increases connection overhead and blocks all rendering until every file is downloaded and executed.

**Is there unused JavaScript?** Yes. Libraries such as KolorTools (virtual tours), venobox, lightbox2, flexslider, and the Google Maps cluster scripts are loaded globally on the homepage even though they are not needed on that page.

**Total JavaScript transferred:** approximately 386 KB encoded across 43 requests (including Maps sub-requests).

---

### CSS

**How CSS is bundled:**
The site uses two strategies in parallel. Two CDN-hosted CSS files are loaded as `<link>` tags (Bootstrap 3.3.5 from `cdn.jsdelivr.net` and the Drupal Bootstrap companion styles from the same CDN). Drupal 7's CSS aggregation has created 11 inline `<style>` blocks in the `<head>`, but each one contains only `@import` statements pointing to individual module CSS files rather than actual inlined rules. The result is 94 CSS resources loaded at runtime via `@import` chains.

**Is this the right decision?** No. Using `@import` inside a wrapper defeats the purpose of aggregation. The browser must fetch the wrapper before it can discover the inner files, adding at least one extra round-trip before any styles are available.

**Is there unused CSS?** Yes, severely. Chrome coverage measured 571 KB of total CSS, of which only 72 KB was used on the homepage — **87% of all CSS is unused** on the first load.

**Source maps:** None.

**Total CSS:** 571 KB decoded, compressed to approximately 192 KB in transit, across 96 requests.

---

### Images

All images are JPEG or PNG — no WebP or AVIF. No `<img>` uses `srcset`. Hero images are served at a fixed 1349×622 px to every viewport. No `loading="lazy"` or `fetchpriority` attributes are used. Drupal image styles provide basic resizing for thumbnails only. One slider image alone transfers ~840 KB.

---

### Third-Party Resources

| Tool                                    | Domain               | Scripts             | Load strategy                  |
| --------------------------------------- | -------------------- | ------------------- | ------------------------------ |
| Google Tag Manager                      | googletagmanager.com | 3 requests          | `async`                        |
| Google Analytics Universal (deprecated) | google-analytics.com | 1 request           | `async`                        |
| Google Analytics GA4 (×2 properties)    | googletagmanager.com | via GTM             | `async` via GTM                |
| Google Maps API                         | maps.googleapis.com  | 3 requests          | **synchronous, blocking**      |
| DoubleClick / Google Ads                | doubleclick.net      | 0 (iframe + beacon) | deferred beacon                |
| Bootstrap CSS + JS                      | cdn.jsdelivr.net     | 2 requests          | `<link>` / blocking `<script>` |

Notable: the deprecated Universal Analytics snippet (`analytics.js`) is still loaded alongside two live GA4 properties. Google Maps is the only third-party that blocks rendering. Bootstrap CDN files carry no Subresource Integrity hash.

---

## Coverage Analysis

### Critical CSS

No critical CSS is extracted or inlined anywhere on the page. The 11 inline `<style>` tags in the `<head>` contain exclusively `@import` statements — they are `@import` wrappers, not actual inlined styles. There are two small inline style blocks injected by the Drupal Builder module for layout overrides, but these are not render-critical styles in the traditional sense. There is no `<link rel="preload">` hint for any stylesheet, font, or script.

### Unused CSS

Chrome DevTools coverage was collected over a full homepage load (desktop, unthrottled).

| Metric            | Value            |
| ----------------- | ---------------- |
| Total CSS decoded | 571 KB           |
| CSS actually used | 72 KB            |
| **Unused CSS**    | **499 KB (87%)** |

Top contributors to unused CSS by waste:

| File                         | Total    | Used   | Waste    |
| ---------------------------- | -------- | ------ | -------- |
| Bootstrap 3.3.5 (CDN)        | 144 KB   | 9.4 KB | 134.6 KB |
| ontt.css (main theme)        | 102.5 KB | 51 KB  | 51.5 KB  |
| animate.css (Builder module) | 72.6 KB  | 0.1 KB | 72.5 KB  |
| Google Fonts — Roboto        | 42.2 KB  | 0 KB   | 42.2 KB  |
| Font Awesome (theme copy)    | 36.5 KB  | 0.6 KB | 35.9 KB  |
| Font Awesome (Builder copy)  | 29.8 KB  | 0.6 KB | 29.2 KB  |
| responsive.css (theme)       | 28.3 KB  | 1.7 KB | 26.6 KB  |
| Google Fonts — Open Sans     | 22.7 KB  | 0 KB   | 22.7 KB  |
| drupal-bootstrap (CDN)       | 15.9 KB  | 1.1 KB | 14.8 KB  |

Key observations:

- **Bootstrap ships 134.6 KB of unused CSS** because only a small fraction of its grid and component rules match elements on the homepage.
- **animate.css (72.5 KB waste)** is loaded by the Builder module but no elements on the homepage have animated classes applied — the library is dead weight.
- **Font Awesome is loaded twice** (theme copy and Builder module copy), and together they waste ~65 KB; only ~1.2 KB of icon rules are matched.
- **Both Google Fonts families (Roboto and Open Sans) are 100% unused** on the homepage, meaning those two stylesheet requests transfer bytes that have zero effect on rendering.

### Unused JavaScript

The JS file sizes recorded during the coverage run (2,807 KB decoded total) reflect the download cost, not execution coverage. Coverage execution data was unreliable for this site because most scripts execute once at page load before coverage listeners attach. However, the following specific libraries are observably unnecessary on the homepage based on DOM inspection:

| Library                                | Size (decoded) | Reason it is unnecessary on the homepage  |
| -------------------------------------- | -------------- | ----------------------------------------- |
| animate.css driver / Builder animation | —              | No animated elements present              |
| lightbox2 (`lightbox.js`)              | 44.6 KB        | No lightbox-linked images on homepage     |
| magnific-popup                         | 42.6 KB        | Redundant with lightbox2; not triggered   |
| flexslider                             | —              | bxslider handles the homepage slider      |
| KolorTools / KolorBootstrap            | —              | Virtual tour feature; no tour on homepage |
| venobox                                | —              | Video modal; not used on homepage         |
| `analytics.js` (UA deprecated)         | 51.1 KB        | UA shut down; data never processed        |
| Bootstrap JS (theme copy)              | 67.3 KB        | Duplicate of CDN Bootstrap JS             |

The three Google Tag Manager / gtag bundles together account for approximately **1,332 KB decoded** (GTM container 359 KB + two gtag payloads ~496 KB + ~472 KB). This is the single largest JS cost on the page, driven entirely by analytics infrastructure.

---

## Flame Chart and Frame Performance

The following observations are based on a DevTools Performance recording of the homepage load and scroll, plus data collected via the browser's performance and coverage APIs.

### Page load

- The browser encounters 37 synchronous, non-deferred `<script>` tags in the `<head>` before it can render anything. Each one pauses the HTML parser. This creates a single extended Long Task during early page load that covers most of the time between TTFB and FCP.
- The Google Maps API itself emits a console warning: _"Google Maps JavaScript API has been loaded directly without `loading=async`. This can result in suboptimal performance."_ — confirming the blocking load pattern.
- Script parse and execution for jQuery, Bootstrap, and the slider libraries all occur inside this blocking window.

### Scroll performance

- The homepage contains a **JavaScript-driven parallax row** (`builder-row-parallax`) implemented by `parallax_background.js`. The element's computed `background-attachment` is `scroll`, not `fixed`, which means the JavaScript library manually repositions the background image on every scroll event by updating an inline style. This pattern forces a **paint on every scroll frame** because changing `background-position` via JavaScript is not a compositor-only operation — it triggers layout recalculation and repaint on the affected layer.
- There are **864 elements with `transition: all`** applied via a CSS rule from the theme or Bootstrap. `transition: all` forces the browser to re-evaluate every animatable CSS property on those 864 elements whenever any property changes — including on hover, focus, or any class toggle. During scroll, if any class is added or removed (as the in-view detection scripts do), this triggers a style recalculation over hundreds of elements.
- Two separate slider auto-play timers (bxslider and flexslider) are both active simultaneously, competing for animation frame budget throughout the page lifetime.

### Interactive actions

- Hovering over navigation links or thumbnails triggers `transition: all` re-evaluation across all 864 affected elements.
- The in-view detection library (`jquery.inview.min.js`) fires a scroll listener on every scroll event; there is no `requestAnimationFrame` throttling visible in the source, which means it can fire multiple times per frame on fast scrolls.

### Dropped or skipped frames

- The JavaScript-driven parallax background and the unthrottled scroll listener together are the most likely cause of dropped frames during scrolling, particularly on lower-powered mobile devices. The field INP of 100 ms is within passing range, but these patterns put it at risk during scroll-triggered interactions.
- These drops are unexpected and avoidable; they are not caused by genuinely complex rendering but by the combination of `transition: all` and JavaScript scroll handlers that bypass the compositor.

---

## Layers and Animations

### Animations

- **Hero slider (bxslider):** The active slider uses `transform: translate3d(x, 0px, 0px)` to move slides — this is a compositor-layer operation and is the correct approach. No first-frame jank is expected from the slide transition itself.
- **Parallax row:** Driven by the Builder module's `parallax_background.js`, which updates an inline `background-position` style on scroll. This is a layout/paint trigger, not a composition trigger. The animation will appear janky on devices where JavaScript cannot update fast enough to match the scroll rate.
- **animate.css:** The `animate.css` library (72.5 KB) is fully loaded but no elements on the homepage carry any `animated` class. The library is inert and contributes no visual animations — only payload waste.
- **Fade effects:** Two elements carry opacity-transition based fade classes. These are driven by CSS `opacity` transitions, which are compositor-safe, but they are wrapped inside `transition: all` rules so the browser still re-evaluates all other properties alongside.

### Paint layers

- 11 elements have an identity CSS matrix transform (`matrix(1,0,0,1,0,0)`) applied, which is a sign of the `transform: translateZ(0)` or `translate3d(0,0,0)` GPU-promotion hack. These are forced composite layers that exist to work around paint issues rather than for a genuine rendering reason.
- Two fixed-position elements (the sticky navigation bar and a scroll-to-top button) each create their own stacking context and paint layer, which is expected and not excessive.
- No `will-change` property is set on any element. This means the parallax row and slider do not get promoted to their own compositor layer in advance, so the browser composites them inline with the main layer and must repaint on every frame during transitions.

### Summary of animation triggers

| Animation                  | Trigger type                            | Compositor-safe?                                         |
| -------------------------- | --------------------------------------- | -------------------------------------------------------- |
| bxslider slide transition  | `transform: translate3d`                | Yes                                                      |
| Parallax background scroll | JS inline style → `background-position` | No — forces repaint                                      |
| Hover/focus transitions    | `transition: all` (864 elements)        | No — forces style recalc                                 |
| Fade-in on scroll          | CSS `opacity` (via `transition: all`)   | Partially — opacity is safe but `transition: all` is not |
| Forced GPU layers (×11)    | `transform: translateZ(0)` hack         | Unnecessary — promotes layers without purpose            |

**How CSS is bundled:**
The site uses two strategies in parallel. Two CDN-hosted CSS files are loaded as `<link>` tags (Bootstrap 3.3.5 from `cdn.jsdelivr.net` and the Drupal Bootstrap companion styles from the same CDN). Drupal 7's optional CSS aggregation has created aggregated wrapper files that use `@import` to pull in individual module CSS files at runtime, resulting in 94 additional CSS resources loaded via `@import`.

**Problems with this approach:**

- CSS `@import` forces sequential discovery: the browser must fetch the aggregated wrapper before it can detect and request the underlying files, adding at least one extra round-trip
- All 94 CSS files are loaded globally on every page regardless of which modules or features are active on that page
- There is no route-level or component-level splitting

**Source maps:** No CSS source maps.

**Is this the right decision?** No. Using `@import` inside an aggregated file defeats the purpose of aggregation. The correct approach is to fully merge and minify individual CSS files into a small number of real bundles, and to remove or defer styles not needed for the initial render.

**Is there unused CSS?** Yes. PageSpeed Insights flagged unused CSS. Because 94 module-level stylesheets are loaded globally, most pages receive CSS rules for modules and features that are not present on that page.

**Total CSS:** approximately 2.99 MB decoded across 96 CSS requests, compressed to roughly 192 KB in transit.

---

### Images

**Format and size:**
All images on the homepage are served in JPEG or PNG format. No images use next-generation formats such as WebP or AVIF.

**Responsive images:**
No `<img>` element uses the `srcset` attribute. Every image has a single fixed source URL regardless of the device viewport or screen pixel density. Hero and banner images are served at a fixed desktop resolution of 1349×622 pixels to all devices including narrow mobile viewports.

**Lazy loading:**
No images carry `loading="lazy"`. All images default to eager loading, including the below-the-fold thumbnail carousels. No image carries a `fetchpriority` attribute to signal rendering priority.

**Full-resolution file exposure:**
Hero and banner images are served directly from `/sites/default/files/` with no access restriction or URL-level protection. Their full-resolution source paths are exposed in the page HTML. One slider image (`2fr_0.jpg`) alone transfers approximately 840 KB.

**Drupal image styles:**
Thumbnail images use Drupal image styles for basic resizing (e.g., `styles/thumb_270_165/`, `styles/thumb_300_300/`). These styles handle width and height cropping, but they produce only JPEG output and do not offer multiple size variants or modern format alternatives.

**Is this the right decision?** No. Serving a 1349×622 JPEG to a 375-pixel-wide mobile viewport wastes most of the transferred bytes and directly worsens LCP on mobile.

---

### Third-Party Resources

| Tool                                    | Domain               | Scripts                         | Load strategy                     |
| --------------------------------------- | -------------------- | ------------------------------- | --------------------------------- |
| Google Tag Manager                      | googletagmanager.com | 3 requests                      | `async`                           |
| Google Analytics Universal (deprecated) | google-analytics.com | 1 request                       | `async`                           |
| Google Analytics GA4 — property 1       | googletagmanager.com | via GTM                         | `async` via GTM                   |
| Google Analytics GA4 — property 2       | googletagmanager.com | via GTM                         | `async` via GTM                   |
| Google Maps API                         | maps.googleapis.com  | 3 requests (incl. sub-requests) | **synchronous, no defer**         |
| DoubleClick / Google Ads (DC-8993070)   | doubleclick.net      | 0 scripts (iframe + beacon)     | deferred beacon                   |
| Bootstrap CSS + JS                      | cdn.jsdelivr.net     | 2 requests                      | `<link>` / synchronous `<script>` |

**Notable observations:**

- **Google Analytics is duplicated.** The page loads the deprecated Universal Analytics library (`analytics.js` / UA-116902750-1) in addition to two separate GA4 tracking IDs (G-6JDRN6682Q and G-BHS8ENZZLM). Universal Analytics has been shut down by Google; this request adds unnecessary weight with no data benefit.
- **Two separate GA4 properties.** Firing two GA4 measurement IDs on the same page is unusual. If both are intentional they should be confirmed as necessary; if either is redundant it should be removed.
- **Google Maps API is synchronous and blocking.** The Maps embed script is loaded with no `defer` or `async`, which halts HTML parsing and blocks all rendering while the script downloads.
- **Bootstrap loaded from an external CDN without SRI.** Bootstrap 3.3.5 CSS and JS are served from `cdn.jsdelivr.net` with no Subresource Integrity (SRI) hash. If the CDN is compromised or unavailable, the page loses its base styles or executes untrusted JavaScript.
- **Estimated third-party impact:** Google Maps is the only third-party tool that directly blocks rendering. GTM, GA, and DoubleClick are all async or beacon-based, but together they account for approximately 8–10 additional network requests and several kilobytes of payload on every page load.
