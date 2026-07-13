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
