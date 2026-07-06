# Findings

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