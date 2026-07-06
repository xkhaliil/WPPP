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