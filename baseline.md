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
