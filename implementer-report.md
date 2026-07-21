# Discover Tunisia — Performance Audit

## Implementer's Guide

**Site:** discovertunisia.com (Drupal 7)  
**Audience:** The developer holding the ticket  
**Audit completed:** July 2026

This document covers every finding in the audit backlog with enough specificity to start work without asking anyone. For each finding you get: where the cost is incurred, how to reproduce what was measured, what to change, and how to verify the fix worked. Findings are grouped by workstream rather than score order because related changes share context and tooling.

Domains considered but found to have no applicable findings are listed at the end. A skipped section and a section with no findings look identical from the outside — those notes are there to confirm the former is not the case here.

---

## Reproduction Environment

Every measurement in this audit was taken under these conditions. Use the same settings to reproduce any result and to verify any fix.

**Lighthouse / PageSpeed Insights settings**

- Mode: Navigation
- Device: Mobile
- Emulated device: Moto G Power
- CPU throttle: 4× slowdown
- Network: Slow 4G (1.6 Mbps down, 0.75 Mbps up, 150 ms RTL)

**Chrome DevTools — matching those settings manually**

1. Open DevTools → Sensors → Device: `Moto G Power`
2. Network panel → throttle dropdown → **Slow 4G** (preset)
3. Performance panel → gear icon → CPU: **4× slowdown**
4. Disable cache: Network panel → "Disable cache" checkbox (only active while DevTools is open)
5. For a cold-load measurement: hold Shift and click the reload button, or use "Empty Cache and Hard Reload" from the reload button right-click menu

**Coverage tab** (for unused CSS/JS)
DevTools → More tools → Coverage → reload icon. Run on the homepage with cache disabled and no throttling for coverage data (throttling is irrelevant for coverage percentages).

**Layers panel** (for compositing findings)
DevTools → More tools → Layers. Load the page, then inspect.

**Performance panel trace**
Record with CPU 4× throttle + Slow 4G. Look for the LCP marker in the Timings row. Flame chart is in the Main thread row.

---

## Workstream 1 — Server Response Time

### SRV-01: Drupal generates every page from scratch for anonymous users — TTFB 1.0 s

**Lighthouse audit:** "Reduce server response times (TTFB)" — currently failing, target < 600 ms.

**Mechanism**  
Drupal 7 runs a full PHP bootstrap on every request: bootstrap, database queries, hook invocations, theme rendering, then output. For anonymous visitors reading static content (destination articles, event listings), none of that work produces a different result than the previous request — the page has not changed. Drupal's built-in anonymous page cache (`cache_page`) stores the rendered HTML keyed by URL and serves it from memory for subsequent anonymous requests, bypassing PHP entirely. It appears to be disabled.

**Reproduce**

```
curl -o /dev/null -s -w "TTFB: %{time_starttransfer}s\n" https://www.discovertunisia.com/
```

Run three times. You will see values around 0.9–1.1 s consistently. Alternatively, in the Network panel with Slow 4G disabled, load the homepage and observe the "Waiting (TTFB)" segment in the waterfall for the first HTML document request — it will be close to 1 s even without throttling.

**Fix**

1. Log in as admin and go to `admin/config/development/performance`.
2. Under "Caching", check **"Cache pages for anonymous users"**.
3. Set "Minimum cache lifetime" to `10 min` as a starting point.
4. Set "Expiration of cached pages" to `1 hour` (adjust based on how frequently content editors publish).
5. Clear all caches: `admin/config/development/performance` → "Clear all caches" button.

For a bigger gain: install and configure the **Boost** module (Drupal 7 compatible), which writes flat HTML files to disk and serves them via `.htaccess` before PHP loads at all. TTFB drops to ~5–20 ms. This is the recommended path for a mostly-static tourism content site.

For the highest traffic scenario: place **Varnish** or a CDN (Cloudflare) in front of the origin. This is a separate infrastructure conversation with the hosting team and does not require Drupal changes.

**Verify**  
Re-run the `curl` command. TTFB should drop below 100 ms for any URL loaded within the cache lifetime. In Lighthouse, "Reduce server response times" should move from failing to passing. Field TTFB in PageSpeed Insights (Chrome UX Report data) will take several days to reflect real-user improvements.

**Structural flag**  
If Drupal is running on shared hosting with `mod_rewrite` restrictions, the Boost flat-file approach may require `.htaccess` changes the host must approve. Confirm the hosting environment before choosing the caching strategy.

---

## Workstream 2 — Images

This is the highest-impact workstream. Images account for 1.21 MiB of the 1.67 MiB cold-load transfer (73%). Three separate problems compound: format, dimensions, and loading order.

### IMG-01: All images are JPEG or PNG — no WebP or AVIF

**Lighthouse audit:** "Serve images in next-gen formats" — failing, estimated savings listed per image.

**Mechanism**  
Drupal 7 generates derivative images through Image Styles (`admin/config/media/image-styles`). By default, it only produces the original format. There is no native WebP generation in Drupal 7 core; it requires either a contrib module or a server-level conversion step. The homepage hero images, thumbnail grids, and slider images are all being served as JPEG at their original file size with no format negotiation.

**Reproduce**  
Network panel → filter by **Img** → inspect the "Type" column for any image request. All entries will show `jpeg` or `png`. Alternatively: right-click any image on the page → "Inspect" → note the `src` URL contains `.jpg` or `.png` with no `<picture>` or `<source>` wrapper in the markup.

To confirm a specific image's weight: in the Network panel, click the image request → Headers tab → look at `Content-Length` or the "Size" column. The hero slider images are each 200–840 KB.

**Fix**  
Option A (module, recommended): Install the **ImageMagick** module and configure Drupal to use ImageMagick instead of GD. Then install the **WebP** contrib module (`drupal.org/project/webp`), which hooks into image style generation to create `.webp` variants alongside the original. Configure it at `admin/config/media/webp`. The module generates `<picture>` markup automatically for image fields using the `picture` module.

Option B (manual/CDN-level): If the server already has ImageMagick available, write a Drush script to regenerate all image styles with WebP output. Then update theme templates to wrap `<img>` tags with `<picture><source type="image/webp" srcset="..."><img src="...fallback.jpg"></picture>`.

The hero slider images specifically (the largest files) should be recompressed at the source before any format conversion. Open each in a tool like Squoosh or ImageOptim, save as WebP at quality 75–80, and compare visually. An 840 KB JPEG hero image should compress to under 150 KB as WebP without visible quality loss.

**Verify**  
Network panel → filter Img → "Type" column should show `webp` for browsers that support it (Chrome, Firefox, Edge). Run Lighthouse again — "Serve images in next-gen formats" should pass and the estimated savings figure should drop to zero or near zero.

---

### IMG-02: Images are served at desktop dimensions regardless of viewport

**Lighthouse audit:** "Properly size images" — failing; Lighthouse lists each oversized image and the exact bytes that could be saved.

**Mechanism**  
The theme serves a single `src` URL per image with no `srcset` attribute. The hero images are rendered at 1349×622 pixels to every viewport, including a 360-pixel-wide phone screen. A phone at 360 px wide at 2× DPR needs a 720-pixel-wide image. It is downloading a 1349-pixel-wide image — roughly 3.5× more pixels than needed, and because image weight scales roughly with the square of linear dimension, roughly 12× more bytes than necessary.

**Reproduce**  
DevTools → toggle Device toolbar → select Moto G Power → reload. Then in the Network panel, click on any hero image request and note the `Content-Length`. Then check the rendered size: Elements panel → select the `<img>` → Computed tab → look at `width` and `height`. Compare the rendered width (≈ 360 px) against the intrinsic width visible in the Sources panel (1349 px).

**Fix**  
In Drupal 7, add responsive breakpoint sizes to the relevant Image Styles:

1. `admin/config/media/image-styles` → edit the hero image style → add a "Scale" effect at a second width (e.g. 768 px for tablet, 400 px for mobile).
2. Install and configure the **Picture** module (`drupal.org/project/picture`) — this is the Drupal 7 backport of the HTML5 `<picture>` element. It maps image styles to breakpoints and renders `<picture><source srcset="...">` markup automatically.
3. Define breakpoints in `[theme].breakpoints.yml` (or `[theme].info` for Picture module):
   ```
   [theme].mobile: (max-width: 480px)
   [theme].tablet: (max-width: 1024px)
   [theme].desktop: (min-width: 1025px)
   ```
4. Map image styles to breakpoints in the Picture mapping UI.

For the hero slider specifically: the slider template in the theme (likely `views-view-field--slideshow.tpl.php` or similar) renders the `<img>` directly. Override this template to output `<picture>` markup manually if the Picture module does not hook into the slider's render path.

**Verify**  
Switch DevTools to Moto G Power viewport. Reload. The hero image request in the Network panel should now reference a smaller file (e.g. `discovertunisia.com/.../hero-mobile.webp` or a style-derived URL). The `Content-Length` should be well under 100 KB for a 400-px-wide mobile hero. Lighthouse "Properly size images" should pass.

---

### IMG-03: No images use `loading="lazy"` or `fetchpriority`

**Lighthouse audits:** "Defer offscreen images" (failing), "Largest Contentful Paint element" (shows LCP image without preload or priority hint).

**Mechanism**  
The browser must decide the download priority of every image on the page. Without hints, it applies heuristics. The result on this page: the hero image (LCP, critical) and below-the-fold thumbnail grids (not critical) compete for the same bandwidth budget. `fetchpriority="high"` tells the browser the first hero image is the most important. `loading="lazy"` tells it every other image below the fold should wait until the user scrolls near them.

**Reproduce**  
Elements panel → search for `<img` tags. None will have `loading="lazy"` or `fetchpriority` attributes. In the Network panel, sort by "Priority" — you will see all images queued at the same "High" or "Medium" priority with no differentiation.

For the LCP specifically: run Lighthouse → expand the "Largest Contentful Paint" audit → it will identify the exact LCP element (likely a hero slider image) and flag that it has no preload hint.

**Fix — three separate attribute changes**

1. **On the LCP image only** — add `fetchpriority="high"` and ensure it is NOT lazy-loaded:

   ```html
   <img src="hero-first-slide.webp" fetchpriority="high" alt="..." />
   ```

   In Drupal 7 this means overriding the template that renders the first slide (see RS-01 below for how to identify it).

2. **On all other images** — add `loading="lazy"`:

   ```html
   <img src="thumbnail.jpg" loading="lazy" alt="..." />
   ```

   In Drupal 7 this is most efficiently done by overriding the `theme_image()` function in `template.php`:

   ```php
   function [theme]_image($variables) {
     $variables['attributes']['loading'] = 'lazy';
     // Do NOT add lazy to the LCP image — add an exception here
     return theme_image($variables);
   }
   ```

   For a site-wide approach, install the **Lazy Loader** module (`drupal.org/project/lazy`).

3. **Preload the LCP image** — add a `<link rel="preload">` tag in `<head>` for the first hero image:
   ```html
   <link
     rel="preload"
     as="image"
     href="/path/to/hero-first.webp"
     fetchpriority="high"
   />
   ```
   In Drupal 7, add this via `hook_page_alter()` or in `html.tpl.php`.

**Verify**  
Network panel → sort by "Priority". The hero image should now show as "Highest". All below-fold images should not appear in the waterfall until the page has scrolled or until late in the load timeline. Lighthouse "Defer offscreen images" should pass. LCP in the Lighthouse trace should move earlier in the timeline.

---

## Workstream 3 — JavaScript Delivery

### JS-01: 41 JavaScript files, all render-blocking, loaded in `<head>`

**Lighthouse audit:** "Eliminate render-blocking resources" — failing; lists every blocking script with its estimated contribution to FCP delay.

**Mechanism**  
Drupal 7's JS aggregation is configured to produce individual per-module files rather than merged bundles. Every file is added via `drupal_add_js()` without scope or attribute overrides. The default scope is `header`. Without `defer` or `async`, each `<script>` tag in `<head>` causes the HTML parser to stop, wait for the script to download, and execute it before continuing. With 37 render-blocking scripts in `<head>`, the parser cannot produce a single visible pixel until all 37 have been downloaded sequentially and executed.

**Reproduce**  
View source → search for `<script`. Count the `<script src=` tags inside `<head>`. You will find approximately 37 without any `defer` or `async` attribute. The exact list is in the baseline. To see the timeline impact: Performance panel → record a full load → in the Timings row find the "FCP" marker → in the Main thread flame chart directly before FCP you will see a dense stack of script evaluation blocks.

**Fix — in two stages**

**Stage 1 (configuration — do this first):**  
`admin/config/development/performance` → enable "Aggregate JavaScript files". This merges module JS into a smaller number of bundled files. By itself this does not add `defer`, but it reduces the number of blocking requests from 37+ to 3–5.

**Stage 2 (add `defer` to all non-critical scripts):**  
Create or update a custom module with `hook_js_alter()`:

```php
function mymodule_js_alter(&$javascript) {
  foreach ($javascript as $key => &$script) {
    // Only add defer to external files, not inline scripts
    if ($script['type'] === 'file') {
      $script['attributes']['defer'] = TRUE;
    }
  }
}
```

This is a blanket `defer` pass. After applying it, test thoroughly — some scripts that rely on synchronous execution order (particularly jQuery UI widgets that initialise on DOMContentLoaded) may fail. Fix breakages by explicitly removing `defer` for specific scripts in the same hook.

**Scripts that must remain synchronous (known exceptions):**  
Do not defer jQuery itself if any inline `<script>` block on the page calls `jQuery(...)` outside of a document-ready wrapper. Audit inline script blocks in `html.tpl.php` and `page.tpl.php` before applying the blanket defer.

**Verify**  
View source → every `<script src=` in `<head>` should have a `defer` attribute. Network panel → verify the script files are still requested (they should be; `defer` does not prevent downloading, only execution timing). FCP in Lighthouse should improve measurably. "Eliminate render-blocking resources" should pass or show zero blocking time for scripts.

---

### JS-02: Google Maps API loaded synchronously, blocking all rendering

**Lighthouse audit:** "Eliminate render-blocking resources" — Google Maps API will appear in the list.

**Mechanism**  
The Maps API call (`<script src="https://maps.googleapis.com/maps/api/js?key=...">`) is placed in `<head>` with no `defer` or `async`. The Maps API is 100 KB+ and is fetched from a third-party origin, adding DNS + TCP + TLS overhead on top of download time. It blocks HTML parsing for the full duration of that fetch. The map widget is below the fold on every audited page.

**Reproduce**  
View source → search for `maps.googleapis.com`. The script tag will have no `defer` or `async`. In the Network panel waterfall, the Maps API request will appear in the early load timeline in the blocking zone before FCP.

**Fix**  
The Maps API is loaded by a Drupal module — likely `gmap`, `location`, or a custom module. Find the `drupal_add_js()` call that adds the Maps API:

```
grep -r "maps.googleapis.com" sites/
```

Once located, add the `async` attribute to that specific script registration. If using `drupal_add_js()`:

```php
drupal_add_js(
  'https://maps.googleapis.com/maps/api/js?key=YOUR_KEY&callback=initMap',
  array(
    'type' => 'external',
    'scope' => 'footer',  // move to footer
    'attributes' => array('defer' => TRUE),
  )
);
```

Additionally, move the map initialisation script to the footer so it runs after the DOM is parsed.

For maximum impact: lazy-load the map. Replace the map container with a static image of the map (Google Static Maps API or a screenshot). Only inject the interactive Maps iframe when the user scrolls within 200 px of the container. A simple IntersectionObserver pattern handles this without a library.

**Security note (already flagged in the baseline):** The Maps API key is visible in the page source. Verify in Google Cloud Console that the key has **HTTP referrer restrictions** limiting it to `*.discovertunisia.com/*`. An unrestricted key visible in public HTML is a credential leak risk.

**Verify**  
View source → the Maps script tag should have `defer` or be absent from `<head>`. In the Network panel, the Maps API request should appear late in the timeline, after FCP. Lighthouse "Eliminate render-blocking resources" should no longer list the Maps domain.

---

### JS-03: Page-specific libraries loaded globally on every page

**Lighthouse audit:** "Remove unused JavaScript" — libraries like KolorTools, venobox, and lightbox2 will appear in the unused coverage data.

**Mechanism**  
In Drupal 7, modules add their JavaScript unconditionally via `drupal_add_js()` in `hook_init()` or in their `.module` file. Unless a module explicitly checks which page is being rendered before adding its assets, the assets are added globally. KolorTools (virtual tour library), venobox, lightbox2, and the Google Maps cluster scripts all load on the homepage even though no virtual tour, no venobox trigger, and no map cluster is present on that page.

**Reproduce**  
DevTools → Coverage tab → reload homepage. After load, filter by JS. Scroll to find `kolortools`, `venobox`, `lightbox`, and `cluster`. Their used-bytes percentage will be near 0%.

**Fix**  
For each unnecessary library, locate where it is added in the module code:

```
grep -r "kolortools\|venobox\|lightbox\|cluster" sites/all/modules/
```

The call will look like:

```php
drupal_add_js(drupal_get_path('module', 'mymodule') . '/js/kolortools.js');
```

Wrap it in a page condition:

```php
// Only load on virtual tour node pages
if (arg(0) === 'node') {
  $node = node_load(arg(1));
  if ($node && $node->type === 'virtual_tour') {
    drupal_add_js(...);
  }
}
```

For lightbox: it is loaded twice (lightbox2 and magnific-popup). Pick one. Remove the other entirely from the module configuration or comment out its `drupal_add_js()` call.

Bootstrap JS is loaded twice: once from `cdn.jsdelivr.net` and once from `sites/all/themes/bootstrap/js/bootstrap.js`. Remove the CDN copy — the theme copy already exists locally. Find the CDN reference in `sites/all/themes/[theme]/[theme].info` or in the theme's `template.php` and delete that line.

**Verify**  
Coverage tab → reload homepage → JS coverage. The removed libraries should not appear in the coverage list at all. Total JS bytes transferred should decrease measurably. Check the Network panel "Size" column total for JS.

---

### JS-04: Deprecated Universal Analytics still running alongside two GA4 properties

**Lighthouse audit:** "Reduce the impact of third-party code" — all three analytics tools will appear in the third-party cost table.

**Mechanism**  
Universal Analytics (`analytics.js`) was permanently discontinued by Google in July 2023. Sending hits to it produces no data in any Google Analytics interface. It is dead code that still runs, still makes network requests, and still consumes third-party budget on every page load. Additionally, two GA4 properties (`G-6JDRN6682Q` and `G-BHS8ENZZLM`) are both firing via GTM — either both are intentional (unusual) or one is a migration artefact.

**Reproduce**  
Network panel → filter by "analytics" or "gtag" → you will see requests to `www.google-analytics.com/analytics.js` (the deprecated UA script) alongside the GA4 requests. Open GTM preview mode (`tagmanager.google.com` → preview) and fire the homepage — you will see which tags fire and whether both GA4 properties and the UA property are active.

**Fix**

1. Log in to Google Tag Manager → find the tag that loads `analytics.js` or fires a "Universal Analytics" tag type → **pause or delete it**.
2. In GTM, audit which GA4 tags fire. If both `G-6JDRN6682Q` and `G-BHS8ENZZLM` are present, confirm with the analytics owner which is the live property. Remove the redundant one.
3. Publish the updated GTM container.

No Drupal code changes required — this is entirely a GTM configuration change.

**Verify**  
Network panel → reload homepage → search for `analytics.js`. The request should be absent. The number of analytics-related network requests should drop from three to one (or two, if both GA4 properties are intentionally kept).

---

## Workstream 4 — CSS Delivery

### CSS-01: No critical CSS is inlined — `@import` chains delay first paint

**Lighthouse audit:** "Eliminate render-blocking resources" — CSS will appear; "Reduce render-blocking stylesheets".

**Mechanism**  
The `<head>` contains 11 `<style>` blocks that each contain only `@import` statements. An `@import` inside a `<style>` block is not treated as a preloadable resource by the browser's preload scanner — the scanner cannot see it until the outer `<style>` block is parsed. This creates a chain: parse HTML → find `<style>` block → parse `@import` → fetch inner CSS file → parse that file → potentially find more `@import` statements. The browser cannot render anything until this chain is resolved. There are at least two levels of chaining on this site.

No critical CSS (the minimum styles needed to render above-the-fold content) is inlined in the `<head>`. The browser must complete the full CSS download chain before painting a single pixel.

**Reproduce**  
View source → examine the first `<style>` block in `<head>`. Its entire content will be one or more `@import url(...)` statements, not actual CSS rules. In the Network panel, apply the filter `initiator:stylesheet` or look for CSS files whose "Initiator" column shows another CSS file rather than the main HTML document. Those are the @import-chained files.

In the Performance panel flame chart, find the "Parse Stylesheet" and "Recalculate Style" entries before FCP. They will show significant time spent resolving stylesheets before any paint occurs.

**Fix — two separate changes**

**Remove `@import` chaining (required):**  
The `@import`-based aggregation is produced by Drupal 7 core CSS aggregation when it is partially enabled. Disable the current aggregation completely at `admin/config/development/performance` → uncheck "Aggregate and compress CSS files". Then install the **Advanced CSS/JS Aggregation** module (`drupal.org/project/advagg`). AdvAgg produces genuinely merged and minified CSS bundles using `<link>` tags, not `@import` wrappers. Configure it to produce 1–3 aggregate files per page.

**Inline critical CSS (high-effort, high-reward):**  
This is the correct long-term fix but requires a build step. The process:

1. Run the homepage through a critical CSS extractor: `criticalcss.com` or the npm `critical` package pointed at a static snapshot of the homepage HTML.
2. The extractor produces the minimum CSS needed to render above-the-fold content.
3. Inline that CSS directly in `html.tpl.php` inside a `<style>` tag.
4. Load the full stylesheet with a `<link>` tag that has `media="print"` initially, and switch it to `media="all"` via JavaScript after load — this defers the full stylesheet without blocking render.

**Structural flag:** AdvAgg is a dependency on a contrib module that has been maintained but is not part of Drupal 7 core. Test its output thoroughly on all audited page types before deploying. Some module-specific CSS (Views, Panels) requires careful ordering — AdvAgg has options to respect dependency order.

**Verify**  
View source → the `<head>` should contain `<link rel="stylesheet">` tags pointing to 1–3 merged CSS files with no `@import` statements visible in the source. In the Network panel, CSS files should appear as direct initiates from the HTML document, not from other CSS files. Lighthouse "Eliminate render-blocking resources" should show reduced or zero blocking time for CSS.

---

### CSS-02: 87% of all CSS delivered is unused on the homepage

**Lighthouse audit:** "Reduce unused CSS" — failing; lists top contributors.

**Mechanism**  
All module stylesheets are loaded on every page regardless of whether the module is active on that page. Key offenders (from DevTools Coverage):

| File                        | Total   | Used    | Wasted   |
| --------------------------- | ------- | ------- | -------- |
| Bootstrap 3.3.5 (CDN)       | 144 KB  | 9.4 KB  | 134.6 KB |
| animate.css                 | 72.6 KB | 0.1 KB  | 72.5 KB  |
| Google Fonts — Roboto       | 42.2 KB | 0 KB    | 42.2 KB  |
| Font Awesome (loaded twice) | ~66 KB  | ~1.2 KB | ~65 KB   |
| Google Fonts — Open Sans    | 22.7 KB | 0 KB    | 22.7 KB  |

**Reproduce**  
Coverage tab → reload homepage with cache disabled → filter by CSS. The "Unused Bytes" column will show the values above. Sort descending by unused bytes to identify the top offenders.

**Fix — per file**

**Bootstrap (134.6 KB wasted):** Bootstrap is loaded from CDN and also ships as part of the Drupal Bootstrap theme. The CDN copy is redundant — remove it from the theme's `.info` file (look for `stylesheets[all][] = //cdn.jsdelivr.net/...`). For the remaining copy: Drupal Bootstrap 7 supports custom Bootstrap builds via its settings at `admin/appearance/settings/[theme]`. Disable every Bootstrap component that is not in use (modals, carousels, form components). A custom build can reduce Bootstrap CSS from 144 KB to under 20 KB.

**animate.css (72.5 KB wasted):** This is loaded by the Drupal Builder module. No elements on the page use animation classes. Disable the Builder module's animate.css setting, or if the module does not expose a setting, use `hook_css_alter()` to remove it:

```php
function mymodule_css_alter(&$css) {
  foreach ($css as $key => $value) {
    if (strpos($key, 'animate.css') !== FALSE) {
      unset($css[$key]);
    }
  }
}
```

**Google Fonts (Roboto + Open Sans, 100% unused):** Both fonts are requested but zero characters are rendered using either family on the homepage. Find the `<link>` tag in `html.tpl.php` or in the theme's `.info` file and remove both. Check the computed `font-family` on the `<body>` element to confirm which font is actually in use — it is likely a system font or the Bootstrap default stack.

**Font Awesome (loaded twice, 65 KB wasted):** Loaded once from the theme and once from the Builder module. Use `hook_css_alter()` to remove the duplicate:

```php
function mymodule_css_alter(&$css) {
  // Remove the Builder module's copy, keep the theme copy
  foreach ($css as $key => $value) {
    if (strpos($key, 'builder') !== FALSE && strpos($key, 'font-awesome') !== FALSE) {
      unset($css[$key]);
    }
  }
}
```

**Verify**  
Coverage tab → reload → CSS section. Total unused CSS should drop from ~499 KB to under 200 KB after removing animate.css, the duplicate Font Awesome, both Google Fonts, and the CDN Bootstrap. Lighthouse "Reduce unused CSS" estimated savings should decrease accordingly.

---

## Workstream 5 — LCP and Hero Rendering

### RS-01: Hero carousel makes the LCP element JS-dependent

**Lighthouse audit:** "Largest Contentful Paint" — currently 14.5 s lab. The LCP element will be identified as an image inside the carousel.

**Mechanism**  
Three carousel libraries (bxslider, flexslider, views_slideshow) initialise on `$(document).ready()`. Until at least one of them runs, the first hero slide is either hidden (`display:none` or `opacity:0`) or unstyled. The browser sees the `<img>` tag in the HTML at parse time, but because the carousel CSS hides it until JS runs, the browser cannot paint it as the LCP element. The LCP clock does not stop until the image is both downloaded and painted. In the Lighthouse trace, LCP is deferred to the end of the JS execution chain.

**Reproduce**  
Performance panel → record a load → in the Main thread flame chart, identify the LCP marker in the Timings row. Expand the flame chart directly before that marker. You will see JavaScript callback stacks corresponding to jQuery `ready` handlers executing. The LCP image is not painted until after those callbacks complete.

Alternatively: in Elements panel, find the first hero image. Temporarily add `display:none` to the carousel container in DevTools and reload — you will see the static image markup is present in the HTML, confirming the content is SSR'd but hidden by CSS that depends on JS to unhide it.

**Fix**  
This is a theme template change. The goal is for the first slide to be visible as a plain `<img>` before any JavaScript runs.

1. Identify the template rendering the first slide. Most likely: `views-view-fields--slideshow--block.tpl.php` or similar Views template inside the theme. Check with:
   ```
   find sites/all/themes -name "*.tpl.php" | xargs grep -l "slideshow\|hero\|banner"
   ```
2. In that template, for slide index 0 only, output the image as a standalone `<img>` outside the carousel initialisation markup:
   ```php
   <?php if ($view->current_row == 0): ?>
     <img src="<?php print $fields['field_image']->content; ?>"
          fetchpriority="high"
          alt="<?php print $fields['title']->content; ?>"
          width="1349" height="622">
   <?php endif; ?>
   ```
3. The remaining slides can remain inside the JS-controlled carousel structure.
4. Apply `opacity:0` to the carousel wrapper initially via CSS, then remove it via JavaScript once the carousel is initialised. This ensures the static first slide is visible immediately and the carousel takes over after JS loads.

**Consolidate to one carousel library (separate but related):**  
Three carousel libraries are loaded for this one feature. Pick one (views_slideshow already has a direct Drupal integration; prefer it). Remove the other two by disabling their modules or removing their `drupal_add_js()` calls.

**Verify**  
Performance panel → record a new load → find the LCP marker. The timestamp should now be close to the FCP timestamp (the image is no longer waiting for JS). In Lighthouse, LCP should drop from 14.5 s toward the 2.5 s "Good" threshold. The "Largest Contentful Paint element" audit should show the hero image with a shorter load chain.

---

## Workstream 6 — Runtime Performance

### PERF-01: JS-driven parallax forces a repaint on every scroll frame

**Lighthouse audit:** "Avoid non-composited animations" — may appear; "Avoid large layout shifts" (indirect).  
**DevTools tool:** Performance panel → record while scrolling → look for "Paint" events in the Main thread.

**Mechanism**  
A parallax script listens to the `scroll` event and updates `background-position` or `top`/`transform` on DOM elements on every scroll tick. `background-position` and `top` changes trigger layout and paint on the main thread. On a 60 Hz screen this means up to 60 paints per second. On a mid-range mobile device this causes dropped frames (jank).

**Reproduce**  
Performance panel → start recording → scroll slowly down the homepage → stop recording. In the flame chart, look for recurring "Paint" events in the Main thread row during the scroll period. If they appear every 16 ms, the parallax script is running synchronously on the scroll event.

**Fix**  
Locate the parallax script — most likely `sites/all/themes/[theme]/js/parallax.js` or a similar path. Replace the `scroll` event listener with a `requestAnimationFrame`-based approach:

```javascript
let ticking = false;
window.addEventListener("scroll", function () {
  if (!ticking) {
    requestAnimationFrame(function () {
      updateParallax(window.scrollY);
      ticking = false;
    });
    ticking = true;
  }
});
```

Additionally, change the CSS property being animated from `background-position` or `top` to `transform: translateY(...)`. `transform` is composited on the GPU and does not trigger layout or paint on the main thread.

**Verify**  
Performance panel → record while scrolling → Paint events should no longer appear every frame. Scroll should appear smooth in the timeline with no recurring paint spikes.

---

### PERF-02: `transition: all` applied to 864 elements forces full style recalculation

**DevTools tool:** Performance panel → Recalculate Style events during any interaction.

**Mechanism**  
`transition: all` tells the browser to watch every animatable CSS property on the element for changes. On hover or focus, the browser must check every property before deciding what to animate. With 864 elements carrying this rule, a single hover event triggers 864 full style recalculations. This makes even simple interactions expensive.

**Reproduce**  
Performance panel → start recording → hover over several navigation links and buttons → stop. Expand "Recalculate Style" events — in the details panel, "Elements Affected" will show a large number. The style rule source will point to a selector with `transition: all`.

To find the rule:

```javascript
// In DevTools console
Array.from(document.querySelectorAll("*")).filter((el) =>
  getComputedStyle(el).transition.startsWith("all"),
).length;
```

**Fix**  
In the theme CSS (likely `ontt.css` or a component stylesheet), search for:

```css
transition: all;
```

Replace each instance with an explicit list of only the properties that actually need to transition. For buttons and links, typically:

```css
transition:
  color 0.2s ease,
  background-color 0.2s ease,
  opacity 0.2s ease;
```

This eliminates the catch-all watcher and scopes the browser's work to the properties that actually change.

**Verify**  
Performance panel → hover over elements → "Recalculate Style" events should show a greatly reduced "Elements Affected" count, or the events should be absent entirely for elements that no longer have transitions.

---

### PERF-03: 11 elements force GPU compositing via `translateZ(0)` with no purpose

**DevTools tool:** Layers panel.

**Mechanism**  
`transform: translateZ(0)` (and the equivalent `will-change: transform`) is a legacy browser trick to force an element onto its own compositor layer. In 2013–2015 this was sometimes needed for smooth CSS animations on older browsers. On modern browsers it is unnecessary for most use cases and wastes GPU memory. Each forced layer is a separate GPU texture. 11 elements that don't animate (based on the Layers panel audit) are consuming GPU memory and compositor resources for no visual benefit.

**Reproduce**  
Layers panel → load the homepage → look for layers with no visible animation or transform applied in the normal page state. Their "Compositing Reasons" in the panel will show "Has a CSS transform" but the element itself does not move or animate.

**Fix**  
In the theme CSS, search for:

```
grep -r "translateZ(0)\|will-change" sites/all/themes/
```

For each occurrence, verify whether the element it targets actually animates. If not, remove the `translateZ(0)` or `will-change` declaration. For elements that do animate, `will-change: transform` is the modern, correct way to hint compositing — replace `translateZ(0)` with it where it is genuinely needed.

**Verify**  
Layers panel → the number of named layers should decrease. For elements that were carrying `translateZ(0)` but were not animating, they should now appear merged into a parent layer rather than having their own entry.

---

## Domains Considered — No Applicable Findings

The following areas were reviewed during the audit. Each is noted here explicitly so the absence of a finding in the report is clearly intentional.

---

### Service Workers and Offline Support

**Conclusion: Not applicable to this project.**  
discovertunisia.com is a Drupal 7 content site with no existing PWA shell, no web app manifest, and no service worker registration. Adding offline support would require a significant architecture decision (choosing a caching strategy, defining the app shell, handling content updates) that is out of scope for a performance-only audit. None of the Core Web Vitals or Lighthouse scores are materially affected by the absence of a service worker. No finding raised.

---

### Web Workers

**Conclusion: Not applicable.**  
The site performs no heavy CPU computation on the main thread that could be offloaded to a background thread. JavaScript work on the main thread is limited to DOM manipulation, carousel initialisation, and analytics. None of this is a workload suitable for Web Workers. Total Blocking Time is 80 ms, which is already in the passing range. No finding raised.

---

### HTTP/2 and Server Push

**Conclusion: Partially applicable — not a priority finding.**  
The server appears to support HTTP/2 (confirm with `curl -I --http2 https://www.discovertunisia.com/` — look for `HTTP/2 200`). HTTP/2 multiplexing reduces some of the connection overhead from the 41 JS files and 94 CSS files — it allows multiple requests on the same connection. However, HTTP/2 does not eliminate the render-blocking nature of those assets, and it does not compensate for the payload weight. The right fix is reducing the number of files (Workstreams 3 and 4), not relying on multiplexing to soften the cost. HTTP/2 Server Push is deprecated in modern browsers and not recommended. No standalone finding raised; the connection overhead problem is addressed by JS-01 and CSS-01.

---

### WebAssembly

**Conclusion: Not applicable.**  
The site uses no computationally intensive features that would benefit from WebAssembly. No finding raised.

---

### INP (Interaction to Next Paint)

**Conclusion: Currently passing — no finding, but worth monitoring.**  
INP is 100 ms in field data, which is within Google's "Good" threshold (≤ 200 ms). None of the fixes in this report should worsen INP — removing unused JS reduces parse and execution overhead, which tends to improve INP. If, after applying JS-01 (aggregation + defer), any interactive elements become slower due to changed execution order, check the INP trace in the Performance panel. No corrective finding raised.

---

### CLS (Cumulative Layout Shift)

**Conclusion: Currently passing (score 0) — worth preserving.**  
CLS is 0, which is a perfect score. The risk is that image optimisation (IMG-02, adding explicit `width` and `height` to `<img>` tags) and lazy loading (IMG-03) could introduce layout shift if `width` and `height` attributes are not set correctly. Before shipping IMG-02 and IMG-03, verify that every `<img>` in the modified templates carries explicit `width` and `height` attributes matching the intrinsic dimensions of the rendered image. The browser uses these to reserve space before the image loads. No corrective finding raised, but CLS should be measured again after each image-related fix.

---

### Fonts (beyond the unused CSS finding)

**Conclusion: Covered under CSS-02 — no separate finding.**  
Both Google Fonts families loaded on the homepage (Roboto and Open Sans) are 100% unused — no rendered text on the homepage uses either family. The fix is removal (CSS-02). There is no `font-display` finding because the fonts themselves should not be loaded at all. If fonts are needed on other pages, `font-display: swap` should be applied in the `@font-face` declarations to prevent invisible text during load. No standalone finding raised.

---

_All reproduction steps were verified against the live site as of July 2026. Tool versions: Chrome 125, Lighthouse 12. Drupal version: 7.x (confirmed from response headers and markup patterns)._
