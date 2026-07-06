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
- The PageSpeed lab metrics were measured with **mobile emulation** using **Emulated Moto G Power** and **Slow 4G throttling**.
- For the assignment requirement about a weak result, the clearest confirmed red score is the **Performance 47** on **La Tunisie toute l'annee**.

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

## Cleaned Baseline Findings

The findings below combine rendering and networking issues into a single cleaned list. Each one is intended to be independently observable.

### Corrective findings

1. **Render-blocking CSS and font delivery delay the first visible paint.**
   - **How does this affect users?** Users wait too long before seeing useful content, so the page feels slow immediately.
   - **Which metric(s) are being affected?** FCP 5.1 s, Speed Index 14.9 s, and indirectly LCP.
   - **What is the cause, or most likely cause?** PageSpeed flagged render-blocking requests and font-display problems, which suggests stylesheets and font assets are delaying initial rendering.
   - **What is the solution, or a likely solution?** Inline critical CSS, defer non-critical CSS and JavaScript, preload key fonts, and use `font-display: swap`.

2. **The homepage LCP element appears too late.**
   - **How does this affect users?** The main hero or featured content appears late, so visitors do not quickly see the most important part of the page.
   - **Which metric(s) are being affected?** LCP 14.5 s in lab data and field LCP 3.2 s.
   - **What is the cause, or most likely cause?** The LCP image or hero content is likely too large, discovered too late, or competing with other resources for bandwidth.
   - **What is the solution, or a likely solution?** Preload the LCP image, compress and resize hero media, serve responsive image variants, and simplify above-the-fold content.

3. **Images dominate the page payload.**
   - **How does this affect users?** Users spend most of their mobile bandwidth downloading images, which makes the page feel heavy and slows visible loading.
   - **Which metric(s) are being affected?** Transfer size, FCP, LCP, Speed Index, and overall Performance.
   - **What is the cause, or most likely cause?** In the throttled mobile measurement, images account for about 1.21 MiB out of 1.67 MiB transferred on the cold load, with little additional compression benefit in transit.
   - **What is the solution, or a likely solution?** Use smaller source files, responsive images, stronger image compression, and lazy loading for non-critical visuals.

4. **The homepage makes too many requests.**
   - **How does this affect users?** High request counts create more network overhead and slow asset discovery, especially on high-latency mobile connections.
   - **Which metric(s) are being affected?** FCP, LCP, Speed Index, and total load time.
   - **What is the cause, or most likely cause?** The throttled mobile baseline still showed 111 cold-load requests, including 33 image requests, 36 JavaScript requests, and 37 CSS requests.
   - **What is the solution, or a likely solution?** Remove unnecessary assets, lazy-load below-the-fold images, and reduce the number of files loaded during the initial view.

5. **There is too much front-end code and too many separate CSS and JavaScript files.**
   - **How does this affect users?** Even when files are compressed, the browser still has to download, parse, and execute more code than necessary.
   - **Which metric(s) are being affected?** Performance score, FCP, Speed Index, and Total Blocking Time.
   - **What is the cause, or most likely cause?** The homepage still loads 36 JavaScript files and 37 CSS files in the throttled mobile baseline, and Lighthouse also flagged unused CSS, unused JavaScript, and legacy JavaScript.
   - **What is the solution, or a likely solution?** Remove unused code, defer non-critical scripts, reduce plugin and library bloat, and consolidate assets where practical.

6. **Rendering work is more expensive than it should be after resources arrive.**
   - **How does this affect users?** The page can feel less smooth during layout and visual updates, especially on lower-powered devices.
   - **Which metric(s) are being affected?** Speed Index, LCP, and overall Lighthouse Performance.
   - **What is the cause, or most likely cause?** PageSpeed flagged forced reflow, missing image dimensions, and non-composited animations, which means the browser is doing unnecessary layout and paint work.
   - **What is the solution, or a likely solution?** Add explicit image dimensions, avoid layout-thrashing scripts, prefer `transform` and `opacity` for animations, and simplify expensive visual effects.

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
