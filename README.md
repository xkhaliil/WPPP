# Personal Project Performance Audit Report

## Chosen Website

- **Name:** Discover Tunisia
- **URL:** https://www.discovertunisia.com/

## Why This Is a Good Audit Candidate

Discover Tunisia is a strong candidate for a performance audit because it mixes several page types and interaction patterns inside the same site. It includes image-heavy landing pages, embedded video, multilingual navigation, search, newsletter signup, a full contact form, and content hubs that appear to depend on older front-end patterns and third-party scripts. That combination makes it useful for analyzing both user-facing performance issues and structural problems such as render-blocking assets, large payloads, and accessibility gaps.

It also satisfies the assignment requirement for weak performance signals. The homepage currently shows a failed mobile Core Web Vitals assessment, and the page [La Tunisie toute l'annee](https://www.discovertunisia.com/la-tunisie-toute-lannee) returned a **red mobile Performance score of 47** in PageSpeed Insights.

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

## Pages To Focus On

1. **Homepage**  
   https://www.discovertunisia.com/  
   This is the most important entry point and sets the baseline for the rest of the audit. It already shows a failed mobile Core Web Vitals assessment, so it is useful for identifying site-wide loading and rendering problems.

2. **Tunisia Live**  
   https://www.discovertunisia.com/tunisia-live  
   This page includes embedded video content and is a good example of media-heavy, interactive content. It is useful for evaluating iframe cost, third-party loading, and how embedded media affects mobile Lighthouse scores.

3. **Contact**  
   https://www.discovertunisia.com/contact  
   This page contains a full contact form and is important because forms add interactivity, validation, and extra client-side work. It also helps evaluate accessibility and usability issues beyond raw speed.

4. **Decouvrir**  
   https://www.discovertunisia.com/decouvrir  
   This is a major discovery hub for the site and likely drives users deeper into destination content. It should be audited because hub pages often combine navigation, hero media, and content previews that can create layout and payload issues.

5. **Media Center**  
   https://www.discovertunisia.com/medias  
   This page appears to be especially image-heavy and is a strong candidate for testing image optimization, caching, lazy loading, and network payload size.

6. **Evenements**  
   https://www.discovertunisia.com/evenements  
   An events page is useful because it often behaves like a frequently updated content listing. It helps test how list-based content, promotional assets, and dynamic presentation affect performance.

7. **La Tunisie toute l'annee**  
   https://www.discovertunisia.com/la-tunisie-toute-lannee  
   This page is important because it already produced a **red mobile Performance score of 47**. It should be a priority page in the audit because it gives the report a clear low-performing example to investigate in depth.

8. **Carthage et Sidi Bou Said**  
   https://www.discovertunisia.com/decouvrir/carthage-et-sidi-bou-said  
   This destination detail page is useful as a representative deep-content page. It helps compare whether performance issues are limited to hubs or continue on content pages with destination-specific imagery and storytelling content.

## Notes

- All scores above were gathered from **mobile** PageSpeed Insights runs on **2026-07-06**.
- For the assignment requirement about a weak result, the clearest confirmed red score is the **Performance 47** on **La Tunisie toute l'annee**.

## Baseline From The Primary Page

The baseline for this project starts with the homepage: https://www.discovertunisia.com/

### Primary page baseline metrics

- **Core Web Vitals Assessment:** Failed
- **Performance:** 56
- **Field data:** LCP 3.2 s, INP 100 ms, CLS 0, FCP 2.9 s, TTFB 1.0 s
- **Lab data:** FCP 5.1 s, LCP 14.5 s, TBT 80 ms, CLS 0, Speed Index 14.9 s

### Key findings

1. **The homepage starts rendering too late.**
   - **How does this affect users?** Users wait too long before they see meaningful content, which makes the site feel slow and less responsive on mobile connections.
   - **Which metric(s) are being affected?** First Contentful Paint (5.1 s), Speed Index (14.9 s), and Largest Contentful Paint (14.5 s).
   - **What is the cause, or most likely cause?** PageSpeed flagged render-blocking requests and slow font loading, which suggests CSS, fonts, and possibly older theme assets are delaying the first visible paint.
   - **What is the solution, or a likely solution?** Inline critical CSS, defer non-critical CSS and JavaScript, preload important fonts, and use `font-display: swap` so text can appear sooner.

2. **The largest visible homepage content loads far too slowly.**
   - **How does this affect users?** The main hero or featured section appears late, so visitors do not quickly see the page's most important content.
   - **Which metric(s) are being affected?** Largest Contentful Paint (14.5 s lab) and the field LCP result (3.2 s), which is already outside the "good" range.
   - **What is the cause, or most likely cause?** PageSpeed identified image-delivery issues and LCP request discovery problems, which usually means the hero image is too heavy, not prioritized, or loaded too late.
   - **What is the solution, or a likely solution?** Compress and resize the main images, serve responsive images, use modern formats such as WebP or AVIF where possible, and preload the LCP image so the browser fetches it earlier.

3. **The page sends too much code and media on the first load.**
   - **How does this affect users?** Large downloads slow the page on mobile networks and make the whole experience feel heavier, especially for first-time visitors.
   - **Which metric(s) are being affected?** Performance score, FCP, LCP, and Speed Index.
   - **What is the cause, or most likely cause?** The homepage had an enormous network payload of about 6,082 KiB, plus PageSpeed reported unused JavaScript, unused CSS, legacy JavaScript, and minification opportunities.
   - **What is the solution, or a likely solution?** Remove unused CSS and JavaScript, minify assets, cut legacy scripts where possible, lazy-load non-essential media, and reduce the amount of content loaded above the fold.

4. **Caching and font delivery are not efficient enough.**
   - **How does this affect users?** Repeat visits do not benefit as much as they should, and users may experience slow text rendering or flashes of invisible text.
   - **Which metric(s) are being affected?** FCP, Speed Index, and indirectly LCP.
   - **What is the cause, or most likely cause?** PageSpeed reported inefficient cache lifetimes and font-display issues, which means some assets are not being reused aggressively enough and fonts are likely blocking text rendering.
   - **What is the solution, or a likely solution?** Increase cache lifetimes for static assets, self-host or subset critical fonts if practical, preload key fonts, and use `font-display: swap`.

5. **Layout and rendering work is still more expensive than it should be.**
   - **How does this affect users?** Even when the page is visible, scrolling and rendering can feel less smooth, especially on lower-powered phones.
   - **Which metric(s) are being affected?** Speed Index, LCP, and overall Lighthouse Performance.
   - **What is the cause, or most likely cause?** PageSpeed flagged forced reflow, missing explicit image dimensions, and non-composited animations, all of which increase browser work during rendering.
   - **What is the solution, or a likely solution?** Add explicit `width` and `height` to images, avoid layout-thrashing scripts, prefer `transform` and `opacity` for animations, and simplify expensive visual effects above the fold.

## Networking Baseline From The Primary Page

These networking stats were measured on the homepage after clearing the browser cache once, then reloading the same page for a soft refresh.

### Homepage networking stats

- **Cold-load requests:** 168
- **Cold-load transferred:** 5,438,056 bytes, about 5.19 MiB
- **Cold-load total resource size:** 6,953,756 bytes, about 6.63 MiB
- **Cold-load encoded size:** 5,390,956 bytes, about 5.14 MiB
- **Approximate compression reduction:** about 22.5% overall from decoded resource size to encoded size

### Soft refresh and caching

- **Soft-refresh requests:** 170
- **Soft-refresh transferred:** 1,429 bytes
- **Reduction due to caching:** 5,436,627 bytes less transferred than the cold load, or about 99.97% less network transfer
- **Cached requests on soft refresh:** 167 of 170 requests had zero network transfer in the browser timing data

### Asset mix

- **Images:** 80 requests, 4,831,892 bytes transferred, about 4.61 MiB
- **JavaScript:** 41 requests, 385,678 bytes transferred, about 0.37 MiB
- **CSS:** 38 requests, 203,715 bytes transferred, about 0.19 MiB
- **Other:** 9 requests, 16,771 bytes transferred

### Compression notes

- **Images:** effectively no additional transfer compression in the browser timing data; encoded and decoded sizes were nearly identical, so the main issue is image weight rather than missing gzip or Brotli
- **JavaScript:** encoded 374,878 bytes vs decoded 1,366,396 bytes, so JavaScript is compressed in transit
- **CSS:** encoded 192,315 bytes vs decoded 690,271 bytes, so CSS is also compressed in transit

### Additional networking findings

1. **Images dominate the homepage payload.**
   - **How does this affect users?** Mobile users spend most of their download budget on images, which slows the first meaningful view of the page and makes the homepage feel heavy on slower connections.
   - **Which metric(s) are being affected?** Total transfer size, Largest Contentful Paint (LCP), First Contentful Paint (FCP), and overall Performance.
   - **What is the cause, or most likely cause?** Images account for about 4.61 MiB out of the 5.19 MiB cold-load transfer, which is roughly 89% of transferred bytes.
   - **What is the solution, or a likely solution?** Resize large images, serve responsive image variants, use more aggressive compression, and avoid loading non-critical homepage imagery above the fold.

2. **The homepage makes too many requests.**
   - **How does this affect users?** A high request count increases connection overhead, slows discovery of important assets, and makes the page more fragile on high-latency mobile networks.
   - **Which metric(s) are being affected?** FCP, LCP, Speed Index, and total load time.
   - **What is the cause, or most likely cause?** The homepage issued 168 requests on a cold load, including 80 image requests plus 79 CSS and JavaScript requests combined.
   - **What is the solution, or a likely solution?** Reduce the number of separate files, combine or eliminate low-value assets, lazy-load below-the-fold images, and remove plugins or widgets that contribute extra requests.

3. **JavaScript and CSS are compressed, but there are still too many front-end assets.**
   - **How does this affect users?** Even when compression helps, many CSS and JavaScript files still create extra network overhead and browser parsing work.
   - **Which metric(s) are being affected?** FCP, Speed Index, Total Blocking Time, and overall Performance.
   - **What is the cause, or most likely cause?** The homepage loads 41 JavaScript files and 38 CSS files. The bytes are compressed well, but the file count is still high and matches the earlier Lighthouse findings about unused CSS and JavaScript.
   - **What is the solution, or a likely solution?** Remove unused code, defer non-critical scripts, reduce dependency bloat, and consolidate styles and scripts where practical.

4. **Caching helps a lot on repeat views, but the first visit is still expensive.**
   - **How does this affect users?** Returning visitors get a much faster refresh, but first-time visitors still pay the full cost of a heavy homepage.
   - **Which metric(s) are being affected?** Cold-load transfer size, FCP, LCP, and perceived first-visit speed.
   - **What is the cause, or most likely cause?** The soft refresh transferred only 1,429 bytes, so caching is working well once assets are stored locally. The bigger issue is the large uncached first load.
   - **What is the solution, or a likely solution?** Keep the strong caching behavior, but reduce the cold-load payload itself by optimizing images, trimming CSS and JavaScript, and prioritizing only the assets needed for the initial viewport.
