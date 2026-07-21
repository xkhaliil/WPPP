# Findings

## Prioritization System

For this project, I will use a custom prioritization model called **WISE**:

- **W**idespread reach: how many homepage users are likely affected
- **I**mpact on user experience: how much the problem hurts perceived performance or usability
- **S**everity of the metric problem: how strongly the issue affects Lighthouse or Core Web Vitals
- **E**ase of implementation: how realistic the fix is to ship without major redesign work

Each category is rated from **1 to 5**.

- **Reach:** `1` = narrow issue, `5` = affects nearly all homepage visitors
- **Impact:** `1` = minor annoyance, `5` = major user-facing slowdown or friction
- **Severity:** `1` = weak metric signal, `5` = strong metric failure or major contributor
- **Ease:** `1` = difficult or high-risk fix, `5` = relatively straightforward fix

### WISE scoring formula

`Final Score = (Impact x 3) + (Severity x 3) + (Reach x 2) + (Ease x 2)`

This gives a maximum score of **50**.

### Priority bands

- **40-50:** High priority
- **30-39:** Medium priority
- **Below 30:** Lower priority

## Cleaned Baseline Findings

The findings below combine rendering and networking issues into a single cleaned list. Each one is intended to be independently observable.

### Corrective findings

1. **Render-blocking CSS and font delivery delay the first visible paint.**
   - **How does this affect users?** Users wait too long before seeing useful content, so the page feels slow immediately.
   - **Which metric(s) are being affected?** FCP 5.1 s, Speed Index 14.9 s, and indirectly LCP.
   - **What is the cause, or most likely cause?** PageSpeed flagged render-blocking requests and font-display problems, which suggests stylesheets and font assets are delaying initial rendering.
   - **What is the solution, or a likely solution?** Inline critical CSS, defer non-critical CSS and JavaScript, preload key fonts, and use `font-display: swap`.
   - **WISE rating:** Impact 4, Severity 4, Reach 5, Ease 4
   - **Final score:** 42/50, **High priority**

2. **The homepage LCP element appears too late.**
   - **How does this affect users?** The main hero or featured content appears late, so visitors do not quickly see the most important part of the page.
   - **Which metric(s) are being affected?** LCP 14.5 s in lab data and field LCP 3.2 s.
   - **What is the cause, or most likely cause?** The LCP image or hero content is likely too large, discovered too late, or competing with other resources for bandwidth.
   - **What is the solution, or a likely solution?** Preload the LCP image, compress and resize hero media, serve responsive image variants, and simplify above-the-fold content.
   - **WISE rating:** Impact 5, Severity 5, Reach 5, Ease 3
   - **Final score:** 46/50, **High priority**

3. **Images dominate the page payload.**
   - **How does this affect users?** Users spend most of their mobile bandwidth downloading images, which makes the page feel heavy and slows visible loading.
   - **Which metric(s) are being affected?** Transfer size, FCP, LCP, Speed Index, and overall Performance.
   - **What is the cause, or most likely cause?** In the throttled mobile measurement, images account for about 1.21 MiB out of 1.67 MiB transferred on the cold load, with little additional compression benefit in transit.
   - **What is the solution, or a likely solution?** Use smaller source files, responsive images, stronger image compression, and lazy loading for non-critical visuals.
   - **WISE rating:** Impact 5, Severity 5, Reach 5, Ease 4
   - **Final score:** 48/50, **High priority**

4. **The homepage makes too many requests.**
   - **How does this affect users?** High request counts create more network overhead and slow asset discovery, especially on high-latency mobile connections.
   - **Which metric(s) are being affected?** FCP, LCP, Speed Index, and total load time.
   - **What is the cause, or most likely cause?** The throttled mobile baseline still showed 111 cold-load requests, including 33 image requests, 36 JavaScript requests, and 37 CSS requests.
   - **What is the solution, or a likely solution?** Remove unnecessary assets, lazy-load below-the-fold images, and reduce the number of files loaded during the initial view.
   - **WISE rating:** Impact 4, Severity 4, Reach 5, Ease 3
   - **Final score:** 40/50, **High priority**

5. **There is too much front-end code and too many separate CSS and JavaScript files.**
   - **How does this affect users?** Even when files are compressed, the browser still has to download, parse, and execute more code than necessary.
   - **Which metric(s) are being affected?** Performance score, FCP, Speed Index, and Total Blocking Time.
   - **What is the cause, or most likely cause?** The homepage still loads 36 JavaScript files and 37 CSS files in the throttled mobile baseline, and Lighthouse also flagged unused CSS, unused JavaScript, and legacy JavaScript.
   - **What is the solution, or a likely solution?** Remove unused code, defer non-critical scripts, reduce plugin and library bloat, and consolidate assets where practical.
   - **WISE rating:** Impact 4, Severity 4, Reach 5, Ease 2
   - **Final score:** 38/50, **Medium priority**

6. **Rendering work is more expensive than it should be after resources arrive.**
   - **How does this affect users?** The page can feel less smooth during layout and visual updates, especially on lower-powered devices.
   - **Which metric(s) are being affected?** Speed Index, LCP, and overall Lighthouse Performance.
   - **What is the cause, or most likely cause?** PageSpeed flagged forced reflow, missing image dimensions, and non-composited animations, which means the browser is doing unnecessary layout and paint work.
   - **What is the solution, or a likely solution?** Add explicit image dimensions, avoid layout-thrashing scripts, prefer `transform` and `opacity` for animations, and simplify expensive visual effects.
   - **WISE rating:** Impact 3, Severity 3, Reach 4, Ease 3
   - **Final score:** 32/50, **Medium priority**

### Ranked corrective backlog (baseline findings only)

1. **Images dominate the page payload**: 48/50
2. **The homepage LCP element appears too late**: 46/50
3. **Render-blocking CSS and font delivery delay the first visible paint**: 42/50
4. **The homepage makes too many requests**: 40/50
5. **There is too much front-end code and too many separate CSS and JavaScript files**: 38/50
6. **Rendering work is more expensive than it should be after resources arrive**: 32/50

---

## Rendering Strategy Findings

### Corrective findings

**RS1. The homepage hero carousel is rendered client-side by JavaScript, making the LCP element JS-dependent.**

- **How does this affect users?** The most prominent visual element on the page — the hero carousel — is not visible until three competing JavaScript carousel libraries have loaded and executed. On a slow mobile connection this means the above-the-fold area is blank or in a broken layout state for the entire duration of the JS loading chain. The lab LCP of 14.5 s and the field LCP of 3.2 s are both rooted directly in this decision. Users on Slow 4G perceive the page as empty for an unacceptably long time even though the server delivered a fully-formed HTML document.
- **Which metric(s) are being affected?** LCP (14.5 s lab / 3.2 s field), FCP (5.1 s lab), Speed Index (14.9 s).
- **What is the cause?** The Drupal theme uses three JS carousel plugins (bxslider, flexslider, views_slideshow) to manage the hero. The first slide is only visible after the carousel JS has initialized and run, which happens after all render-blocking scripts in `<head>` have been downloaded and executed. The HTML markup for the carousel is present in the SSR output, but without the carousel JS the images are either hidden or unstyled — so SSR's advantage of delivering content immediately is negated at exactly the most visible point on the page.
- **What is the solution?** Render the first hero slide as a plain server-side HTML `<img>` element that is immediately visible without any JavaScript. Add `fetchpriority="high"` to that image. Use JavaScript only to add slide navigation for subsequent frames, and load it with `defer`. Remove two of the three carousel libraries and consolidate onto one.
- **Is this the best rendering choice?** No. The first slide's content is fully static and known at request time. There is no reason for it to depend on JavaScript for its initial display. Server-rendered first slide with JS-enhanced interactivity is the correct pattern.
- **WISE rating:** Impact 5, Severity 5, Reach 5, Ease 3
- **Final score:** 46/50, **High priority**

---

**RS2. Drupal 7 serves all pages with uncached dynamic SSR, producing a 1.0 s TTFB for static content.**

- **How does this affect users?** TTFB is 1.0 s on mobile. For a tourism content site where the vast majority of pages contain fully static content — destination articles, event listings, media galleries — regenerating the full HTML on every anonymous request wastes server time and creates a visible delay before the browser can begin parsing any HTML. Because nothing can render before the first byte of HTML arrives, every millisecond of unnecessary TTFB directly delays both FCP and LCP.
- **Which metric(s) are being affected?** TTFB (1.0 s field), FCP (5.1 s lab), LCP (14.5 s lab / 3.2 s field), overall Performance score.
- **What is the cause?** Drupal 7's page cache for anonymous users appears to be disabled or unconfigured. Without caching, every page view triggers a full PHP bootstrap, multiple database queries, Drupal hook invocations, and theme rendering — all before a single byte is sent to the browser. The content on most pages does not change between requests, so this full rebuild is unnecessary for anonymous visitors.
- **What is the solution?** Enable Drupal's built-in anonymous page cache at `admin/config/development/performance`. For higher-traffic scenarios, place a reverse-proxy cache (Varnish) or CDN (Cloudflare Page Cache) in front of the origin. For the lowest-traffic static pages, the Boost module can export flat HTML files that bypass PHP entirely. Any of these options would reduce TTFB to approximately 50–100 ms from the current 1.0 s.
- **Is this the best rendering choice?** No. For publicly accessible, largely static content that does not change on every request, full dynamic uncached SSR on every hit is the least efficient option available within the existing Drupal architecture. A cached SSR response requires no architectural change and is straightforward to enable.
- **WISE rating:** Impact 4, Severity 4, Reach 5, Ease 4
- **Final score:** 42/50, **High priority**

---

### Updated ranked corrective backlog (including rendering strategy findings)

| Rank | ID  | Finding                                                             | Score |
| ---- | --- | ------------------------------------------------------------------- | ----- |
| 1    | —   | Images dominate the page payload                                    | 48/50 |
| 2    | B1  | JS delivered as 41 individual synchronous blocking files            | 46/50 |
| 2    | B3  | Images have no next-gen format, srcset, or lazy loading             | 46/50 |
| 2    | —   | The homepage LCP element appears too late                           | 46/50 |
| 2    | RS1 | Hero carousel is CSR-dependent — LCP gated behind JS execution      | 46/50 |
| 6    | C1  | No critical CSS inlined — @import chains delay first paint          | 44/50 |
| 7    | —   | Render-blocking CSS and font delivery delay the first visible paint | 42/50 |
| 7    | B4  | Google Maps API loads synchronously, blocking the main thread       | 42/50 |
| 7    | RS2 | Dynamic uncached SSR on every request produces 1.0 s TTFB           | 42/50 |
| 10   | —   | The homepage makes too many requests                                | 40/50 |
| 10   | B2  | CSS fragmented across 94 @import-loaded files, no route splitting   | 40/50 |
| 10   | C2  | 87% of all CSS delivered to the browser is unused                   | 40/50 |
| 13   | —   | Too much front-end code and too many separate JS/CSS files          | 38/50 |
| 14   | F1  | JS-driven parallax triggers a repaint on every scroll frame         | 37/50 |
| 15   | F2  | `transition: all` on 864 elements forces full style recalculation   | 36/50 |
| 16   | C3  | JS payload dominated by ~1,332 KB of analytics infrastructure       | 34/50 |
| 17   | —   | Rendering work is more expensive than it should be                  | 32/50 |
| 17   | B5  | Duplicate and deprecated analytics (UA + 2× GA4)                    | 32/50 |
| 19   | L1  | animate.css (72.5 KB) loaded but zero elements use it               | 28/50 |
| 20   | L2  | 11 forced GPU layers via `translateZ(0)` hack without purpose       | 26/50 |

---

## Bundle-Specific Findings

### Corrective findings

**B1. JavaScript is delivered as 41 individual synchronous render-blocking files with no bundling.**

- **How does this affect users?** The browser cannot begin rendering any visible content until it has downloaded and executed every one of the 41 script files loaded in the `<head>`. On a slow mobile connection this adds a significant delay before the first pixel appears.
- **Which metric(s) are being affected?** FCP (5.1 s lab), LCP (14.5 s lab), Speed Index (14.9 s), and Total Blocking Time (80 ms).
- **What is the cause?** Drupal 7's JS aggregation is disabled or misconfigured. Every module ships its own JavaScript file and all of them are included unconditionally in the `<head>` with no `defer` or `async` attribute. Libraries that are only needed on specific pages — KolorTools (virtual tours), venobox, lightbox2, flexslider — are also loaded on the homepage. Bootstrap 3.3.5 JS is loaded twice: once from the CDN and once from the Drupal Bootstrap theme.
- **What is the solution?** Enable Drupal's built-in JS aggregation to produce a small number of concatenated, minified bundles. Add `defer` to all scripts that do not need to run during initial parsing. Move page-specific libraries to be loaded only on the pages that require them. Remove the duplicate Bootstrap JS.
- **WISE rating:** Impact 5, Severity 5, Reach 5, Ease 3
- **Final score:** 46/50, **High priority**

---

**B2. CSS is fragmented across 94 @import-loaded files and is not split by route or component.**

- **How does this affect users?** Because all 94 CSS files are discovered sequentially through `@import` chains, the browser cannot start downloading the inner files until it has already fetched the outer aggregated wrappers. This adds at least one extra network round-trip before render-critical styles are available, worsening FCP and perceived paint speed.
- **Which metric(s) are being affected?** FCP (5.1 s lab), Speed Index (14.9 s), and the render-blocking resource audit in PageSpeed Insights.
- **What is the cause?** Drupal 7's CSS aggregation is set to produce "aggregated" files that still use `@import` to pull in each module's CSS file individually, rather than fully merging and inlining the rules. Additionally, all module stylesheets are loaded on every page even when the corresponding module is not active, so there is no per-route scoping. Bootstrap 3.3.5 CSS is also loaded from an external CDN without a Subresource Integrity hash.
- **What is the solution?** Disable `@import`-based aggregation and replace it with a build step (or a Drupal module such as Advanced CSS/JS Aggregation) that produces genuinely merged and minified CSS bundles. Audit each module's stylesheet and restrict loading to the pages where it is actually needed. Add an SRI hash to the CDN Bootstrap link, or self-host Bootstrap to remove the external dependency.
- **WISE rating:** Impact 4, Severity 4, Reach 5, Ease 3
- **Final score:** 40/50, **High priority**

---

**B3. Images have no next-gen format support, no responsive sizes, and no lazy loading.**

- **How does this affect users?** Mobile users receive the same 1349×622-pixel JPEG files that desktop users receive, even though their viewport is less than 400 pixels wide. Most of the bytes downloaded for each hero image are wasted. No images below the fold are deferred, so the browser competes for bandwidth between visible and invisible images simultaneously.
- **Which metric(s) are being affected?** LCP (14.5 s lab / 3.2 s field), FCP, Speed Index, and total page weight (images currently account for ~1.21 MiB of the 1.67 MiB cold-load transfer).
- **What is the cause?** The Drupal theme serves all images with a single `src` URL and no `srcset`. Drupal image styles are used for thumbnail crops but are not configured to generate multiple responsive sizes or WebP/AVIF variants. No `loading="lazy"` attribute is applied to below-the-fold images, and no `fetchpriority="high"` is set on the LCP image.
- **What is the solution?** Configure Drupal image styles to generate WebP variants alongside JPEG fallbacks and serve them using `<picture>` with `<source type="image/webp">`. Define at least two or three responsive width breakpoints for each image style so mobile devices receive smaller files. Add `loading="lazy"` to all below-the-fold images. Add `fetchpriority="high"` to the first hero/LCP image only.
- **WISE rating:** Impact 5, Severity 5, Reach 5, Ease 3
- **Final score:** 46/50, **High priority**

---

**B4. Google Maps API is loaded synchronously, blocking the main thread before it is visually needed.**

- **How does this affect users?** The Google Maps script is loaded with a plain blocking `<script>` tag in the `<head>`, which pauses HTML parsing until the Maps library is fully downloaded and executed. The map widget is not visible above the fold, so this delay is paid before any user-facing content has rendered.
- **Which metric(s) are being affected?** FCP, LCP, Speed Index, and Total Blocking Time.
- **What is the cause?** The Drupal module responsible for embedding the Google Maps API places the script tag without a `defer` or `async` attribute. The embedded API key is also directly visible in the page HTML, which is a security concern if the key lacks API restrictions in Google Cloud Console.
- **What is the solution?** Add `defer` to the Google Maps script tag so it loads in parallel with HTML parsing and executes after the document is parsed. Consider lazy-loading the map only when the user scrolls near the map widget. Verify that the Maps API key has appropriate HTTP referrer and API-level restrictions applied in Google Cloud Console.
- **WISE rating:** Impact 4, Severity 4, Reach 5, Ease 4
- **Final score:** 42/50, **High priority**

---

**B5. Duplicate and deprecated analytics tools waste third-party budget on every page load.**

- **How does this affect users?** Loading three separate analytics tools (deprecated Universal Analytics `analytics.js`, plus two GA4 properties) means the browser makes additional network requests, parses more JavaScript, and sends more beacons on every page load. This is a recurring cost for every visitor even though Universal Analytics data is no longer being processed by Google.
- **Which metric(s) are being affected?** Total network transfer, Total Blocking Time, and overall third-party payload.
- **What is the cause?** When GA4 was adopted, the old Universal Analytics snippet was not removed. Two separate GA4 measurement IDs (G-6JDRN6682Q and G-BHS8ENZZLM) are also both active, suggesting either a migration artefact or an unreviewed duplication.
- **What is the solution?** Remove the deprecated Universal Analytics snippet (`analytics.js` / UA-116902750-1) entirely. Audit the two GA4 properties and confirm whether both are intentional; remove or consolidate whichever is redundant. Consolidate all analytics firing through a single GTM container to keep third-party script count as low as possible.
- **WISE rating:** Impact 2, Severity 2, Reach 5, Ease 5
- **Final score:** 32/50, **Medium priority**

---

### Updated ranked corrective backlog (including bundle findings)

| Rank | Finding                                                      | Score |
| ---- | ------------------------------------------------------------ | ----- |
| 1    | Images dominate the page payload                             | 48/50 |
| 2    | The homepage LCP element appears too late                    | 46/50 |
| 2    | B1: JS is 41 individual synchronous blocking files           | 46/50 |
| 2    | B3: Images have no next-gen format, srcset, or lazy loading  | 46/50 |
| 5    | Render-blocking CSS and font delivery delay first paint      | 42/50 |
| 5    | B4: Google Maps API loads synchronously                      | 42/50 |
| 7    | The homepage makes too many requests                         | 40/50 |
| 7    | B2: CSS is 94 @import-loaded files with no route splitting   | 40/50 |
| 9    | There is too much front-end code and too many separate files | 38/50 |
| 10   | Rendering work is more expensive than it should be           | 32/50 |
| 10   | B5: Duplicate and deprecated analytics                       | 32/50 |

---

## Master Ranked Backlog (All Findings)

This table merges all corrective findings across all audit rounds — baseline, bundle, coverage, flame chart, layers, and rendering strategies.

| Rank | ID  | Finding                                                             | Score | Priority |
| ---- | --- | ------------------------------------------------------------------- | ----- | -------- |
| 1    | —   | Images dominate the page payload                                    | 48/50 | High     |
| 2    | B1  | JS delivered as 41 individual synchronous blocking files            | 46/50 | High     |
| 2    | B3  | Images have no next-gen format, srcset, or lazy loading             | 46/50 | High     |
| 2    | —   | The homepage LCP element appears too late                           | 46/50 | High     |
| 2    | RS1 | Hero carousel is CSR-dependent — LCP gated behind JS execution      | 46/50 | High     |
| 6    | C1  | No critical CSS inlined — @import chains delay first paint          | 44/50 | High     |
| 7    | —   | Render-blocking CSS and font delivery delay the first visible paint | 42/50 | High     |
| 7    | B4  | Google Maps API loads synchronously, blocking the main thread       | 42/50 | High     |
| 7    | RS2 | Dynamic uncached SSR on every request produces 1.0 s TTFB           | 42/50 | High     |
| 10   | —   | The homepage makes too many requests                                | 40/50 | High     |
| 10   | B2  | CSS fragmented across 94 @import-loaded files, no route splitting   | 40/50 | High     |
| 10   | C2  | 87% of all CSS delivered to the browser is unused                   | 40/50 | High     |
| 13   | —   | Too much front-end code and too many separate JS/CSS files          | 38/50 | Medium   |
| 14   | F1  | JS-driven parallax triggers a repaint on every scroll frame         | 37/50 | Medium   |
| 15   | F2  | `transition: all` on 864 elements forces full style recalculation   | 36/50 | Medium   |
| 16   | C3  | JS payload dominated by ~1,332 KB of analytics infrastructure       | 34/50 | Medium   |
| 17   | —   | Rendering work is more expensive than it should be                  | 32/50 | Medium   |
| 17   | B5  | Duplicate and deprecated analytics (UA + 2× GA4)                    | 32/50 | Medium   |
| 19   | L1  | animate.css (72.5 KB) loaded but zero elements use it               | 28/50 | Lower    |
| 20   | L2  | 11 forced GPU layers via `translateZ(0)` hack without purpose       | 26/50 | Lower    |

### Good findings

1. **Layout stability is already strong.**
   - **How does this affect users?** The page does not visibly jump around while loading, which makes reading and tapping more reliable.
   - **Which metric(s) are being affected?** CLS 0.
   - **What is the cause, or most likely cause?** The page appears to reserve space well enough for major content and avoids large unexpected shifts during loading.
   - **What is the solution, or a likely solution?** Preserve this behavior while optimizing images and layout by keeping explicit dimensions and stable content containers.

2. **Repeat-view caching is very effective.**
   - **How does this affect users?** CSS and JavaScript files already benefit from transfer compression, which reduces the cost of downloading front-end code on mobile networks.
   - **Which metric(s) are being affected?** Transfer size, FCP, and overall network efficiency.
   - **What is the cause, or most likely cause?** CSS and JavaScript encoded sizes are substantially smaller than decoded sizes, showing that compression is already enabled in transit.
   - **What is the solution, or a likely solution?** Preserve this compression setup while focusing future work on reducing file count and image weight.

## Mobile-Specific Findings

1. **Corrective: Slow 4G mobile conditions still expose a heavy first-view experience.**
   - **How does this affect users?** Mobile users on constrained networks still face a noticeably heavy first load before the homepage feels complete.
   - **Which metric(s) are being affected?** Mobile Performance 56, FCP 5.1 s, LCP 14.5 s, Speed Index 14.9 s, and cold-load mobile transfer of about 1.67 MiB.
   - **What is the cause, or most likely cause?** The homepage still combines too many requests with too much image weight for a throttled mobile connection.
   - **What is the solution, or a likely solution?** Prioritize lighter above-the-fold imagery, fewer initial requests, and more aggressive deferral of non-critical assets for mobile users.

2. **Good: Mobile responsiveness and layout stability are already strong.**
   - **How does this affect users?** Once the page is visible, mobile users are less likely to experience laggy taps or frustrating layout shifts.
   - **Which metric(s) are being affected?** INP 100 ms and CLS 0.
   - **What is the cause, or most likely cause?** The page does not appear to suffer from major interaction delays or unstable layout movement during the measured mobile run.
   - **What is the solution, or a likely solution?** Keep protecting these strengths while optimizing load performance so improvements to speed do not introduce new instability.

---

## Coverage Findings

### Corrective findings

**C1. No critical CSS is extracted or inlined — the browser cannot render anything until all @import chains resolve.**

- **How does this affect users?** Because zero render-critical styles are inlined in the HTML, the browser must complete at least two sequential network round-trips (fetch the wrapper `<style>` tag → fetch each `@import`-ed file) before it can paint a single pixel. On a slow mobile connection this is the direct cause of FCP 5.1 s and Speed Index 14.9 s.
- **Which metric(s) are being affected?** FCP (5.1 s lab), Speed Index (14.9 s), and indirectly LCP.
- **What is the cause?** Drupal 7 has no built-in critical-CSS extraction. The 11 inline `<style>` tags in the `<head>` are produced by Drupal's CSS aggregation system and contain only `@import` URLs — they hold no actual style rules. No `<link rel="preload">` hints compensate for this.
- **What is the solution?** Extract the minimal above-the-fold CSS rules (primarily typography, layout skeleton, and header styles — roughly 5–10 KB) and inline them in a `<style>` tag. Move all remaining stylesheets to load asynchronously using `rel="preload"` + `onload` swap or the `media="print"` trick. This can be done with a Drupal module such as Critical CSS or as a post-processing step in a build pipeline.
- **WISE rating:** Impact 5, Severity 5, Reach 5, Ease 2
- **Final score:** 44/50, **High priority**

---

**C2. 87% of all CSS delivered to the browser is unused on the homepage.**

- **How does this affect users?** The browser downloads, parses, and builds a CSSOM from 571 KB of stylesheets, of which 499 KB (87%) contains rules that do not match any element on the homepage. This parse work happens before the browser can render and directly inflates FCP and Total Blocking Time.
- **Which metric(s) are being affected?** FCP (5.1 s lab), Speed Index (14.9 s), and Total Blocking Time (80 ms).
- **What is the cause?**
  - **Bootstrap 3.3.5** ships 144 KB of CSS; only 9.4 KB is matched on the homepage (93% waste). All carousel, modal, panel, and form component styles are downloaded even though those components are not on the homepage.
  - **animate.css** (72.6 KB) is loaded by the Builder module but zero animation classes are applied to any homepage element — the entire file is dead weight.
  - **Font Awesome is loaded twice** (theme copy 36.5 KB + Builder module copy 29.8 KB = 66.3 KB total) with only ~1.2 KB of icon rules matched.
  - **Both Google Fonts families (Roboto and Open Sans) are 100% unused** on the homepage; their CSS `@import` requests spend bandwidth with no visual result.
  - **responsive.css** contributes 28.3 KB of which 26.6 KB is unused.
- **What is the solution?** Remove animate.css and both duplicate Font Awesome copies immediately. Replace the Google Fonts imports with self-hosted fonts scoped only to the families and weights that are actually used. Run PurgeCSS or a PostCSS uncss step against Bootstrap to ship only the components that appear on each page. Split the theme's `responsive.css` into per-breakpoint or per-page chunks.
- **WISE rating:** Impact 4, Severity 4, Reach 5, Ease 3
- **Final score:** 40/50, **High priority**

---

**C3. The JavaScript payload is dominated by analytics infrastructure that does not serve the user.**

- **How does this affect users?** The three Google Tag Manager / gtag bundles together decode to approximately 1,332 KB — more than the entire image payload on the soft-refresh view. Even though these scripts are loaded `async`, they compete with first-party scripts for CPU time during the critical load window and on low-powered mobile devices they extend the period during which the main thread is busy.
- **Which metric(s) are being affected?** Total Blocking Time, INP, and overall JS parse time.
- **What is the cause?** Three separate gtag entry points are loaded (two GA4 properties plus a DoubleClick Floodlight tag), each pulling its own full copy of the gtag runtime. The deprecated `analytics.js` Universal Analytics library (51.1 KB) is also still present even though Universal Analytics data collection ended in 2023.
- **What is the solution?** Consolidate all tracking under a single GTM container with a single gtag runtime. Remove `analytics.js` entirely. Evaluate whether both GA4 properties are necessary and remove redundant ones. Consider server-side tagging to move analytics payload off the client entirely.
- **WISE rating:** Impact 3, Severity 3, Reach 5, Ease 3
- **Final score:** 34/50, **Medium priority**

---

## Flame Chart and Frame Drop Findings

### Corrective findings

**F1. A JavaScript-driven parallax background triggers a repaint on every scroll frame.**

- **How does this affect users?** During scrolling, the parallax section causes the browser to repaint rather than composite, which drops frames and makes the page feel janky on mid-range and low-end mobile devices. The field INP of 100 ms is still within the passing threshold, but this pattern directly threatens it under real-world scroll load.
- **Which metric(s) are being affected?** INP (100 ms field), perceived scroll smoothness, and potentially CLS if repaints cause layout shifts.
- **What is the cause?** The Builder module's `parallax_background.js` updates a `background-position` inline style on every scroll event. Changing `background-position` is not a compositor-only operation — it invalidates the paint record for the affected layer and forces a repaint each frame. The element's `background-attachment` is `scroll` (not `fixed`), confirming the JavaScript is doing manual repositioning without `will-change` or compositor promotion. Additionally, the scroll listener is not wrapped in `requestAnimationFrame`, so it can fire multiple times per frame on fast scrolls.
- **What is the solution?** Replace the JavaScript parallax with a CSS-only approach using `background-attachment: fixed` (which the browser composites natively) or a `transform: translateY()` on a child element driven by a `requestAnimationFrame`-throttled listener. Alternatively, remove the parallax effect entirely for mobile viewports where it provides minimal visual benefit and maximum performance cost.
- **WISE rating:** Impact 4, Severity 3, Reach 5, Ease 3
- **Final score:** 37/50, **Medium priority**

---

**F2. `transition: all` is applied to 864 elements, forcing full style recalculation on every interaction.**

- **How does this affect users?** Whenever any CSS property changes on any of the 864 affected elements — including on hover, focus, class toggle, or scroll-triggered class addition by the in-view library — the browser must re-evaluate every animatable CSS property on all 864 elements simultaneously. On mobile this manifests as a brief but perceptible stutter during navigation hover states and in-view reveals.
- **Which metric(s) are being affected?** INP (100 ms), scroll smoothness, and interaction responsiveness.
- **What is the cause?** The theme CSS (likely inherited from Bootstrap 3 or the `ontt` theme) applies a blanket `transition: all` rule. The Chrome coverage run found 864 elements with `transitionProperty === 'all'` at page load. `transition: all` is a known anti-pattern: it tells the browser to animate every property, which prevents the rendering pipeline from optimising which properties actually change.
- **What is the solution?** Replace `transition: all` with explicit property declarations such as `transition: color 0.2s ease, background-color 0.2s ease` targeting only the properties that are actually animated. Audit the theme and Bootstrap overrides for the source rule and remove or narrow it. This is a pure CSS change with no visual regression risk if the specific properties are preserved.
- **WISE rating:** Impact 3, Severity 3, Reach 5, Ease 4
- **Final score:** 36/50, **Medium priority**

---

## Layers and Animations Findings

### Corrective findings

**L1. animate.css (72.5 KB) is fully loaded but no element on the homepage uses it.**

- **How does this affect users?** The browser downloads and parses 72.5 KB of CSS animation keyframe definitions that produce zero visual output on the homepage. This is pure payload waste that delays CSSOM construction.
- **Which metric(s) are being affected?** FCP, Speed Index, and total CSS weight.
- **What is the cause?** The Drupal Builder module unconditionally loads `animate.css` as a module dependency on every page, regardless of whether any content block on that page uses animated entry effects. Chrome coverage confirmed 0.1 KB used out of 72.6 KB.
- **What is the solution?** Configure the Builder module to load `animate.css` only on pages where animated blocks are present, or replace the blanket `@import` with a conditional attachment in the module's `.info` file. If animated blocks are not used on the site at all, remove the dependency entirely.
- **WISE rating:** Impact 2, Severity 2, Reach 5, Ease 4
- **Final score:** 28/50, **Lower priority**

---

**L2. Eleven elements use the `transform: translateZ(0)` GPU-promotion hack without a genuine rendering reason.**

- **How does this affect users?** Each forced composite layer consumes GPU memory. On mobile devices with limited VRAM, excessive layers cause the GPU to thrash, which leads to dropped frames during scroll and animation. These 11 forced layers do not correspond to any animated element, so they provide no rendering benefit.
- **Which metric(s) are being affected?** Scroll smoothness and memory pressure, particularly on low-end mobile.
- **What is the cause?** Eleven elements carry an identity CSS transform (`transform: matrix(1,0,0,1,0,0)`) with no `will-change` property. This pattern — applying `transform: translate3d(0,0,0)` or `translateZ(0)` as a hack to force GPU compositing — was common in older jQuery-era theme code and libraries (such as bxslider and older Bootstrap versions). The transforms are identity matrices, meaning they have no visual effect but they do promote the elements to separate compositor layers.
- **What is the solution?** Audit the theme CSS and slider library CSS for `transform: translateZ(0)` or `translate3d(0,0,0)` declarations not associated with actual animations. Remove the ones that are purely promotional hacks. For the slider, rely on the `transform: translate3d(x, 0, 0)` that is applied dynamically during slide transitions — this is legitimate — but remove any static identity transform that is added as a baseline style.
- **WISE rating:** Impact 2, Severity 2, Reach 4, Ease 3
- **Final score:** 26/50, **Lower priority**
