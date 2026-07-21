# Discover Tunisia — Performance Audit
## Executive Briefing

**Site:** discovertunisia.com  
**Audience:** Leadership, product ownership, budget holders  
**Audit completed:** July 2026

---

## In Brief

The Discover Tunisia website has a measurable, documented mobile performance problem. The homepage fails Google's official quality standard for mobile users. Two additional content pages score in Google's red performance band. On a real mobile connection, a visitor waits more than three seconds before the main hero image appears — the precise threshold at which, according to Google's own research, more than half of mobile users leave.

These are not hypothetical risks or benchmark exercises. They affect visitors arriving on the site today and they influence how the site ranks in Google Search.

The foundations of the site are sound. Content is correctly indexed, the site loads without visual instability, and returning visitors have a noticeably faster experience. The performance problems are real, but they are concentrated in a small number of fixable areas. This report explains what they are, what they cost, and which ones to address first.

---

## How to Verify These Numbers Yourself

Every metric in this report comes from two free, public sources. You do not need a developer to use either of them.

**Google PageSpeed Insights** → [pagespeed.web.dev](https://pagespeed.web.dev)  
Enter any page URL. Google runs a live test and returns the same scores, labels, and pass/fail verdicts used throughout this report. The results are stable and reproducible. If the homepage shows a "Failed" Core Web Vitals badge and a Performance score in the orange or red band on mobile, the diagnosis this report describes is confirmed.

**Your own phone**  
Open discovertunisia.com on a mobile device using a cellular data connection — not WiFi. Note how long it takes before the main hero image is visible. Then run the same URL on PageSpeed Insights. The two experiences will match.

---

## What Is Already Working

A report that lists only failures is a hostile document, and a defensive reader does not act on recommendations. The site has real strengths worth acknowledging — and worth protecting if optimization work begins.

| Area | Measurement | What it means in practice |
| --- | --- | --- |
| **Search visibility** | SEO score: 100 / 100 | The site is correctly structured for Google indexing. This is difficult to preserve after major rebuilds and should be treated as a hard constraint on any future work. |
| **Layout stability** | CLS: 0 (perfect score) | Nothing shifts or jumps unexpectedly as the page loads. Users are not accidentally tapping the wrong element. A CLS of zero is uncommon on content-heavy pages. |
| **Responsiveness to taps** | INP: 100 ms (passing) | When a user taps a button or link, the site responds quickly. Interactive elements are not the bottleneck. |
| **Security and best practices** | Best Practices: 96 / 100 | The site follows current browser security guidance. |
| **Repeat visit speed** | 33% less data transferred on second visit | The server caching layer works for returning visitors. Someone who reloads the homepage gets a noticeably lighter page. |

These are meaningful. The goal of any optimization work should be to improve the failing areas without disturbing these passing ones.

---

## The Problems — In Business Terms

### Problem 1 — Visitors on mobile wait too long before they see anything useful

> On a real mobile connection, the main hero image on the homepage takes **3.2 seconds** to appear. In Google's standardised test environment, that same image takes **14.5 seconds**.

**Why 3.2 seconds matters:**  
Google published research showing that 53% of mobile visits are abandoned when a page takes longer than three seconds to load. Discover Tunisia's hero image arrives at exactly that threshold in real-world field data. Every visitor who arrives on a mobile connection and does not see the main image within three seconds is a visitor who is statistically likely to leave before engaging with any content.

**Why 14.5 seconds matters:**  
The gap between the real-world number (3.2 s) and the test number (14.5 s) is large. That gap signals a structural problem rather than a marginal one. The test environment simulates a mid-range Android phone on a standard mobile connection — the most common device class among mobile users globally. The 14.5-second result means the current architecture is performing near its ceiling in real-world conditions; there is little headroom left before real users start experiencing the lab number.

**The cause, in plain terms:**  
The hero image on the homepage is controlled by three separate JavaScript animation libraries. The page has to download and run all three before the image is visible — even though the image itself is on the server and ready to send. The content is instant; the technology displaying it is not. Removing that dependency is the single most impactful technical fix in this report.

**Who is affected:** Every visitor who arrives on the homepage from a mobile device. According to global web usage data, mobile accounts for roughly 60% of web traffic. For a tourism-oriented site targeting individual travellers, that share is likely higher.

---

### Problem 2 — Three content pages are in Google's red performance band

> Google scores pages on a 0–100 scale. Below 50 is "Poor" (red). Between 50 and 89 is "Needs Improvement" (orange). Above 90 is "Good" (green). The site currently has no pages in the green band on mobile.

| Page | Mobile Score | Google's Band |
| --- | --- | --- |
| La Tunisie toute l'annee | **47** | 🔴 Poor |
| Tunisia Live | **52** | 🟠 Needs Improvement |
| Homepage | **56** | 🟠 Needs Improvement |

**Why this affects search ranking:**  
Google confirmed in 2021 that Core Web Vitals — the measurements behind these scores — are a ranking signal in Search. A page that consistently fails these measurements may rank lower than a competing page that passes them, assuming content quality is otherwise comparable. For a tourism content site competing on destination search terms ("things to do in Tunisia", "Tunisia beaches", etc.), a red or orange performance score is a direct structural disadvantage against competitors whose pages score green.

**Why La Tunisie toute l'annee is the most urgent individual page:**  
At 47, it sits in the red band. It almost certainly carries destination content that competes directly in search for users who are actively researching Tunisia trips — the highest-value audience for this site. A red score is the strongest signal Google sends that a page is underperforming. It should be included in any first-phase optimization scope alongside the homepage.

---

### Problem 3 — Every visitor downloads far more than they need

> The homepage transfers **1.67 MB** of data on a first visit. Of all the styling code loaded, **87% is never used** on that page. Several tools loaded on every visit serve no active purpose.

**Why page weight matters for this site's audience:**  
In Tunisia and across North Africa, mobile data plans are frequently metered. A 1.67 MB homepage has a tangible cost per visit for users on pay-per-MB plans. Beyond direct cost, raw weight slows loading on every visit — not only the first one.

**Where the weight comes from:**  
Images alone account for 1.21 MB of the 1.67 MB total — 73% of everything the browser downloads on a first visit. They are served at desktop resolution to every device, including phones. No images use modern compressed formats (WebP or AVIF). Converting to modern formats and sending appropriately sized images to mobile screens would reduce the image payload by an estimated 60–70%.

**Specific waste that is straightforward to remove:**

- A full animation library (72.5 KB) is loaded on every page, but no animations are currently active anywhere on the site. It is dead weight.
- A virtual-tour JavaScript library is loaded on the homepage, even though virtual tours only appear on specific destination pages.
- Two separate lightbox image-viewer tools are loaded on every page. One would do the same job.
- A deprecated Google analytics tool (Universal Analytics, officially discontinued by Google in July 2023) is still running alongside two live analytics systems. All three run simultaneously on every page load, tripling the analytics overhead.

None of these were deliberate choices. They are the accumulated result of plugins and tools added over time that were never audited for continued necessity. The cost is borne by every visitor.

---

## The Risk of Inaction

| Risk | Current status | Potential consequence |
| --- | --- | --- |
| **Search ranking** | Core Web Vitals: **Failed**. Google's ranking signal is active. | Reduced organic visibility for destination search terms; increased dependence on paid acquisition to maintain traffic. |
| **Mobile abandonment** | LCP at 3.2 s — at the industry abandonment threshold | A measurable share of mobile visitors leave before seeing any destination content. |
| **Competitive disadvantage** | No pages pass the green band; competing tourism sites that do pass will rank above equivalent pages | Structural and growing. Competitors investing in performance gain ground that is increasingly expensive to recover. |
| **Technical debt compounding** | Each new plugin or module adds to the weight with no counterbalancing audit | Future optimization becomes more expensive. The longer the delay, the larger the remediation scope. |

---

## Recommended Investment Roadmap

The findings in this audit group naturally into three tiers. The ranking reflects business impact first and implementation risk second.

---

### Tier 1 — High impact, low structural risk
*Configuration changes and asset cleanup. No redesign. No new platform.*  
**Recommended timeline: start immediately.**

| Action | Business outcome | Honest effort |
| --- | --- | --- |
| **Enable server-side page caching** | Cuts the 1.0-second server response delay to under 0.1 seconds for all anonymous visitors. A configuration change, not a code change. | 1–2 days |
| **Compress and resize images for mobile** | Images are 73% of what visitors download. Modern formats and mobile-appropriate sizes reduce page weight by an estimated 60–70% on mobile devices. | 1–2 weeks |
| **Remove unused code and dead libraries** | Eliminates the animation library, the virtual-tour script, the duplicate lightbox tool, and the deprecated analytics snippet — together several hundred kilobytes removed from every page load. | 3–5 days |
| **Prioritise the La Tunisie toute l'annee page** | Red score (47) on a high-value destination page. Include in the same scope as the homepage to move both pages out of the red band. | Covered by the above actions |

**Expected outcome from Tier 1:** Page weight reduction of approximately 50%, TTFB improvement from 1.0 s to under 0.1 s, improved scores across all audited pages.

---

### Tier 2 — High impact, moderate development effort
*Requires front-end development work. No platform change required.*  
**Recommended timeline: following Tier 1.**

| Action | Business outcome | Honest effort |
| --- | --- | --- |
| **Display the first hero image without depending on JavaScript** | The most direct fix for the slow LCP. The first slide becomes immediately visible on page load, before any script runs. | 1–2 weeks |
| **Tell the browser to prioritise the hero image** | A one-line addition that signals the browser to load the most important visual first. Disproportionate impact for minimal effort. | 1–2 days |
| **Defer the Google Maps script** | Maps currently blocks the entire page from rendering, even on pages where no map is visible. Moving it to load after the page is displayed removes the blockage. | 1–2 days |
| **Consolidate three carousel libraries into one** | Removes conflicting code and reduces the script overhead for every page that uses a slider. | 1–2 weeks |

**Expected outcome from Tier 2:** LCP should move below 2.5 seconds (Google's "Good" threshold), FCP improvement, potential Core Web Vitals pass on the homepage.

---

### Tier 3 — Structural improvements
*Platform-level decisions. Higher cost and longer timeline, but addresses root causes.*  
**Recommended timeline: medium-term planning.**

| Action | Business outcome | Honest effort |
| --- | --- | --- |
| **Audit and retire inactive Drupal modules** | The CSS and JavaScript bloat originates largely from active modules that load their assets on every page regardless of whether the module is in use. Removing unused modules is the cleanest long-term solution. | 2–4 weeks |
| **Add a CDN or edge delivery layer** | Reduces load time globally rather than just at the server origin. Protects against traffic spikes and improves experience for international visitors. | 1–2 weeks (infrastructure) |

---

## What to Fund First

**The single most valuable investment is Tier 1** — specifically image optimization and server caching, in that order.

Image optimization addresses the largest single contributor to page weight and has a direct effect on LCP, the metric that is currently failing Core Web Vitals. It applies to every page, every visitor, every device, and can be delivered without any redesign risk.

Server caching costs almost nothing to implement and eliminates a 1.0-second delay that currently affects every anonymous visitor to the site. It is a configuration change.

**La Tunisie toute l'annee** should be in scope for the first phase. A red score on a high-value destination page is a real and present search ranking risk.

**What not to do:** A full platform migration or redesign is not necessary and is not recommended at this stage. All high-priority findings are addressable within the existing Drupal architecture. A redesign would introduce significant SEO risk, reset the site's accumulated authority, and would not by itself resolve the performance problems unless the same optimization work were done alongside it.

---

## Summary

| Finding | Business risk | Tier | Effort |
| --- | --- | --- | --- |
| Hero image takes 3+ seconds on mobile | User abandonment, LCP failure | 2 | Medium |
| La Tunisie toute l'annee scores red (47) | Search ranking disadvantage | 1 | Low–Medium |
| 1.67 MB page weight, 73% images | Data cost, slow load, LCP impact | 1 | Low–Medium |
| Server regenerates every page from scratch | Unnecessary 1.0 s delay on every visit | 1 | Low |
| Unused libraries loaded on every page | Wasted load time, maintenance overhead | 1 | Low |
| Google Maps blocks page rendering | FCP and LCP delay | 2 | Low |
| Deprecated analytics tool still running | Wasted third-party budget, redundant data | 1 | Low |

---

*All metrics were measured using Google PageSpeed Insights (mobile, Emulated Moto G Power, Slow 4G throttling) and Google Chrome DevTools. Field data (LCP, INP, CLS, FCP, TTFB) comes from the Chrome User Experience Report — real measurements from real users visiting the site. Audit date: July 2026.*
