---
title: "VeSync Smart Scale: Integration Feasibility for Bryan Health Stack"
type: research_brief
date: 2026-03-30
requested_by: kishan
tags: [health, smart-scale, vesync, etekcity, integration, BIA, apple-health]
mission: null
sources_count: 12
summary: "VeSync (Etekcity) smart scales offer reliable body weight sync to Apple Health and MyFitnessPal, but have no official API, limited pyvesync library support for scales, intermittent Apple Health sync bugs, and BIA-derived body composition metrics (fat %, muscle mass, hydration) that are directional at best — integration into the local health stack requires an Apple Health bridge, not a direct API connector."
---

## background

"V-Sync smart scale" most likely refers to **VeSync** — the app/cloud platform operated by Vesync Co., Ltd (also the parent brand behind Levoit, Cosori, and Etekcity). The Etekcity scale lineup connects exclusively via the VeSync app. Key models in the ecosystem:

- **Etekcity ESF24** — base Bluetooth BIA scale, 13 measurements, ~$30–40
- **Etekcity FIT 8S** — WiFi-enabled, 12 measurements, VeSync cloud sync
- **Etekcity HR Smart Fitness Scale** — adds heart rate via grip sensors, ~$80

All models use Bioelectrical Impedance Analysis (BIA) via foot electrodes. All are managed through the free VeSync app (iOS/Android). There is no separate "VSync" scale brand on the market — the "V" may have been misread from "VeSync."

If kishan is referring to a different brand (e.g., a local/international brand with a similar name), that ecosystem was not found in research. The Etekcity/VeSync interpretation is the only credible match.

## key findings

- **No official VeSync developer API.** Vesync Co. publishes no public API, OAuth flow, or developer documentation for data retrieval. There is no personal access token system analogous to Oura. Source: VeSync developer docs search (none found); pyvesync library documentation, https://webdjoe.github.io/pyvesync/latest/

- **pyvesync Python library does NOT support scales.** The `pyvesync` library (github.com/webdjoe/pyvesync) is the primary community reverse-engineering of the VeSync API. Supported device types: outlets, switches, bulbs, fans, air purifiers, humidifiers, air fryers, thermostats. Scales are explicitly absent from the supported device list. A GitHub issue (#56, opened 2020) requested scale support and identified the `/cloud/v1/deviceManaged/fatScale/getWeighData` API endpoint via packet capture — but the feature was never merged and the issue is stale. Source: https://webdjoe.github.io/pyvesync/latest/supported_devices/ and https://github.com/webdjoe/pyvesync/issues/56

- **Apple Health sync exists but is unreliable.** Etekcity officially advertises VeSync → Apple Health sync for weight, body fat %, BMI, and other metrics. However, App Store reviews as recently as March 2026 report that body fat % and BMI fail to write correctly to the Health app after iOS/app updates, even when VeSync appears as an authorized data source in Health settings. Body weight sync appears more consistently reliable than body composition metrics. Sources: https://apps.apple.com/us/app/vesync/id1289575311 (recent reviews); Amazon Q&A for Etekcity scales.

- **MyFitnessPal sync: body weight only.** Official MFP integration via VeSync app is confirmed, but MFP's support documentation explicitly states only body weight syncs — not body fat %, BMI, or other composition metrics. Source: https://support.myfitnesspal.com/hc/en-us/articles/4413373710349-Etekcity-VeSync

- **Fitbit sync has reported breakage.** Reddit reports (July 2024) of VeSync ↔ Fitbit sync silently stopping after working initially. Source: https://www.reddit.com/r/fitbit/comments/1edq7if/vesync_not_syncing_with_fitbit/ (single-sourced, anecdotal)

- **Google Fit and Samsung Health** are listed as supported integrations on Etekcity product pages. No reliability data found in research.

- **BIA body composition accuracy is limited.** Multiple independent sources — including a BodySpec meta-analysis of smart scale validation studies and two PMC peer-reviewed studies — report that BIA body fat % estimates carry ±3–8 percentage-point error compared to DEXA, increasing with higher adiposity. Lean mass / muscle mass estimates show even larger errors. Hydration estimates are circular: BIA measures hydration via electrical resistance, which is directly confounded by hydration state at the time of measurement. Body weight measurement itself is accurate (<1 kg error) and consistent. Sources: https://www.bodyspec.com/blog/post/best_smart_scales_of_2025_expert_recommendations; https://pmc.ncbi.nlm.nih.gov/articles/PMC10622934/; https://pmc.ncbi.nlm.nih.gov/articles/PMC6042744/

- **Local stack integration requires an Apple Health bridge.** The existing health stack at `~/projects/life-data/health` uses direct REST API connectors (Oura PAT → JSON → SQLite). VeSync has no equivalent. The only realistic programmatic path into the local stack is: **VeSync app → Apple Health → Apple Health XML export or a bridge tool (e.g., `healthkit-to-sqlite`, Health Auto Export iOS app) → ingest into SQLite.** This is a multi-hop path with manual or semi-automated steps, not a clean REST connector like Oura. Sources: health stack README; pyvesync library scope.

## source tensions

- **Apple Health sync reliability** is contested: Etekcity marketing claims full integration; App Store reviews (as recently as March 2026) report body fat/BMI not actually writing to Health on current iOS versions. The gap between advertised and working sync is unclear — may be model-dependent, iOS-version-dependent, or intermittent. This is the most significant uncertainty.

- **BIA accuracy** is consistently reported as "useful for trends, not absolute values" across independent reviewers and peer-reviewed literature. Manufacturer materials (Etekcity, Arboleaf) claim "accurate" results without disclosing error ranges. Peer-reviewed literature and third-party review sites consistently qualify this — flag manufacturer accuracy claims as promotional.

- **pyvesync scale endpoint** was identified via packet capture in 2020 (issue #56). It is possible an unofficial integration could be built by reverse-engineering the API directly, but this is unvalidated, undocumented, and fragile. Not a stable path.

## integration verdict

| Data field | Reliability | Usable for Bryan? |
|---|---|---|
| Body weight | High (BIA weight is accurate) | Yes — primary use case |
| Body fat % | Low–medium (±3–8pp vs DEXA) | Trend direction only |
| Lean/muscle mass | Low (BIA-derived) | Not recommended for tracking |
| BMI | High (calculated) | Low utility; weight is better |
| Hydration | Very low (self-confounded) | No |
| Visceral fat, bone density, protein | Very low (model extrapolations) | No |

**Integration path feasibility: buy-with-caveats.**

Body weight alone justifies the purchase — it fills a real gap in the Bryan health stack (current baselines note tracking started 2026-03-29 with manual entry at 151.3 lbs). A scale would automate daily weight capture. The integration path (VeSync → Apple Health → healthkit-to-sqlite or Health Auto Export → SQLite) is achievable but requires a bridge step; it is not a clean API connector. Apple Health sync reliability has open questions based on recent user reports. BIA body composition metrics should be treated as directional trend indicators only — not ground truth.

## related vault nodes

- `health/baselines.md` — Bryan weight baseline (151.3 lbs, 2026-03-29); establishes the tracking gap this scale would fill.
- `system/decisions/001-health-platform-scope.md` (in project repo) — source-agnostic connector architecture; scale ingestion would add a new connector following that pattern.

---

## sources

1. Etekcity ESF24 product page — https://etekcity.com/products/smart-fitness-scale-esf24
2. Etekcity HR Smart Fitness Scale — https://etekcity.com/products/hr-smart-fitness-scale
3. VeSync FIT 8S store listing — https://us.vesync.com/product-detail/etekcity-fit-8s-smart-fitness-scale-18
4. pyvesync supported devices — https://webdjoe.github.io/pyvesync/latest/supported_devices/
5. pyvesync GitHub issue #56 (scale support request) — https://github.com/webdjoe/pyvesync/issues/56
6. VeSync App Store reviews — https://apps.apple.com/us/app/vesync/id1289575311
7. MyFitnessPal / VeSync integration docs — https://support.myfitnesspal.com/hc/en-us/articles/4413373710349-Etekcity-VeSync
8. Reddit: VeSync not syncing with Fitbit — https://www.reddit.com/r/fitbit/comments/1edq7if/vesync_not_syncing_with_fitbit/
9. BodySpec smart scale meta-analysis 2025 — https://www.bodyspec.com/blog/post/best_smart_scales_of_2025_expert_recommendations
10. PMC: BIA vs DEXA reliability study — https://pmc.ncbi.nlm.nih.gov/articles/PMC10622934/
11. PMC: BIA vs DXA body composition comparison — https://pmc.ncbi.nlm.nih.gov/articles/PMC6042744/
12. PCMag Etekcity HR Smart Scale review — https://www.pcmag.com/reviews/etekcity-hr-smart-fitness-scale
