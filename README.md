# ActionLens — From Complaints to Action (AI for Civic Impact)

> **Civic Hackathon Prototype**: An AI-powered civic action engine that converts scattered, duplicate citizen complaints into prioritized, explainable, and trackable municipal interventions across four stages: **Capture → Understand → Reveal → Act**.

---

## 1. Problem Summary

In cities worldwide, municipal complaint portals become black holes:
1. **Complaint Fatigue & Noise**: Citizens file hundreds of uncoordinated, emotional grievances about the same issue using varied colloquial vocabulary (Hinglish, slang, typos).
2. **Duplication Chaos**: Dispatchers waste resources sending three different teams to the same open manhole or waterlogged road.
3. **Black-Box Skepticism**: Citizens have zero visibility into why their complaint isn't resolved, while authorities lack an explainable rationale to prioritize life-threatening emergencies over routine annoyances.
4. **Siloed Symptoms vs. Systemic Root Causes**: A blocked drain, a road pothole, and an overflowing dumpster on the same street are treated as three unrelated tickets rather than one systemic corridor crisis.

**ActionLens** bridges this gap through a 100% deterministic, explainable, in-browser AI pipeline paired with live spatial DBSCAN clustering and decision-maker action briefs.

---

## 2. Quick Setup & Run Instructions

```bash
# 1. Install dependencies
npm install

# 2. Run local development server (runs on port 3000)
npm run dev

# 3. Run unit tests
npx vitest run

# 4. Production build
npm run build
```

---

## 3. Architecture Diagram

```
+---------------------------------------------------------------------------------------+
|                                    ACTIONLENS                                         |
+---------------------------------------------------------------------------------------+
                                           |
         +---------------------------------+---------------------------------+
         |                                                                   |
   [CITIZEN PORTAL]                                                  [AUTHORITY COMMAND]
         |                                                                   |
  1. CAPTURE                                                          3. REVEAL
   - 60s Natural Text Filing                                           - KPI Metrics Row
   - GPS Map Pin & Photo Upload                                        - Priority Queue (0-100)
   - "Let AI Decide" Category                                          - Leaflet Spatial Map
         |                                                             - Hotspot Clusters
         v                                                                   |
  2. UNDERSTAND (AI Pipeline in /src/ai/)                                    v
   +---------------------------------------+                          4. ACT
   | a. Keyword-Weighted Categorization    |                           - 1-Click "Why?" Explanation
   | b. Extractive 1-Sentence Summary      |                           - Merge / Split Duplicates
   | c. TF-IDF + Haversine Deduplication   |                           - Dispatch Fictional Crews
   | d. 6-Factor Explainable Priority      |                           - Decision-Maker Action Briefs
   +---------------------------------------+                           - Citizen Status Tracker
```

---

## 4. AI Pipeline Methodology & Mathematical Formulas

All AI functions are pure, unit-tested, and located in `/src/ai/`:

### Step 1: Categorization (`categorize(text)`)
- Multi-domain keyword dictionary with weights (1 to 5) and synonyms (including Hinglish terms like *kachra*, *waterlog*, *nala*, *live wire*, *manhole*).
- Categories: `Waterlogging/Drainage`, `Road Damage`, `Garbage/Waste`, `Streetlight/Electrical`, `Water Supply`, `Public Safety Hazard`, `Other`.
- Confidence formula:
  $$\text{Confidence} = \min\left(0.97, \max\left(0.60, \text{base} + \frac{S_{\text{top}} - S_{\text{second}}}{S_{\text{top}}} \times 0.15\right)\right)$$
- If the user manually picks a category that differs from the AI classification, both are surfaced with an amber conflict flag.

### Step 2: Extractive Summarization (`summarize(text)`)
- Deterministic extractive algorithm bounded to **max 20 words**.
- Extracts key hazard markers, location phrases, and main verb clauses.
- Optional LLM-assisted enhancement supported via `VITE_LLM_API_KEY` with graceful local fallback. Every output carries a badge: `"Local rules + TF-IDF"` or `"LLM-assisted"`.

### Step 3: Deduplication Engine (`detectDuplicates(newReport, existingReports)`)
- Combines lexical vector similarity with geospatial distance:
  $$\text{Score} = 0.6 \times \text{TextSimilarity} + 0.3 \times \text{ProximityScore} + 0.1 \times \text{CategoryScore}$$
- **TextSimilarity**: Cosine similarity of Term Frequency (TF) vectors with stop-word removal.
- **ProximityScore**: Spherical Haversine distance within a 150m radius:
  $$\text{ProximityScore} = \max\left(0, 1 - \frac{\text{Distance}}{150\text{m}}\right)$$
- **Threshold**: Scores $\ge 0.55$ trigger `"Likely Duplicate"`.
- Citizens can choose to *"Support existing report instead"*, boosting the master ticket's priority and preventing database clutter.

### Step 4: Explainable Prioritization (`prioritize(report, context)`)
Every report receives a score from 12 to 99 and a plain-language *"Why this ranks here"* factor list:
1. **Direct Safety Hazard Keywords** (live wire, open manhole, accident, fire, child, collapse): up to **35 pts**
2. **Complaint Volume & Recurrence** (linked duplicate count $\times 5$): up to **20 pts**
3. **High-Footfall Zone** (school, hospital, market, metro, transit junction): up to **15 pts**
4. **Category Baseline Weight** (Public Safety = 10, Drainage = 8, Road = 7, etc.): up to **10 pts**
5. **Aging Backlog SLA Penalty** ($>7$ days unresolved = 10 pts, $>3$ days = 6 pts): up to **10 pts**
6. **Geographic Hotspot Compounder** (part of an active spatial cluster): up to **10 pts**
7. **Time & Weather Urgency** (*rain*, *night*, *dark*, *flooding*): up to **5 pts**

Levels: **Critical** (75–100), **High** (50–74), **Medium** (30–49), **Low** (<30).

### Step 5: Spatial Clustering (`detectClusters(reports)`)
- Density-based grouping ($\varepsilon \approx 220\text{m}$, minimum 3 open reports).
- Computes centroid coordinates, spatial boundary radius, dominant overlapping categories, and human-readable cluster names (e.g. *"Waterlogging & Drains + Road Damage near Station Road"*).

### Step 6: Decision-Maker Action Brief (`generateActionBrief(cluster)`)
- Generates a 60-second executive summary: Headline, What's Happening, Why It Matters, Recommended First Action, Suggested Department, and a **Root Cause Hypothesis** (explicitly marked *"Hypothesis, not a confirmed fact"*).

---

## 5. Synthetic Dataset Notes

The app is pre-seeded with **60 realistic synthetic reports** located in `/src/data/seed.json`:
- **Geographic Center**: Urban transit corridor around Bangalore Central coordinates (`12.9725, 77.5952`).
- **Safety Hazards (5 Critical)**:
  1. Open manhole uncovered on pedestrian walkway near Nilgiri Market (`CMP-1001`).
  2. Exposed live electric wire dangling near St. Jude School (`CMP-1004`).
  3. Fallen banyan tree blocking dual carriage arterial road (`CMP-1006`).
  4. Railway underpass flooded over 4 feet with partially submerged vehicle (`CMP-1007`).
  5. Broken pedestrian bridge railing over storm canal (`CMP-1008`).
- **8 Deliberate Duplicate Groups**: Worded differently with typos, Hinglish, and varied citizen perspectives.
- **1 Major Systemic Cluster**: 9 reports along Station Road / Central Bazaar combining drain blockage, overflowing garbage, and deep road craters.
- **Ages**: Ranging from 2 hours old to 14 days old (demonstrating SLA escalation).

---

## 6. 5-Minute Guided Demo Script (For Judges)

1. **Step 1 (Start Guided Demo)**:
   - Click the **"Guided Demo"** button in the top navbar.
   - It pre-fills a realistic citizen complaint on Station Road: *"Water logged near the school gate on Station Road, drain blocked since 2 days, bikes stalling continuously!"*
2. **Step 2 (Submit & Watch AI Engine)**:
   - Click **"Submit Complaint & Run AI Analysis"**.
   - Watch the animated 4-stage pipeline verify the category, generate an extractive summary, compute a priority score of **78 (Critical)**, and identify a duplicate match with existing ticket `CMP-1010` (31m away).
   - Copy or note your tracking code (e.g. `CMP-1018`).
3. **Step 3 (Inspect Authority Dashboard)**:
   - Switch role to **"Authority"** via the top toggle.
   - See the KPI row update in real time.
   - Click the **"Why this ranks here?"** expander on the top row to see the exact numerical point breakdown.
   - Click **"Manage"** to open the side drawer: review the photo preview, check the audit log, assign the issue to a crew (*"SWD Rapid Dewatering Gang #1"*), or test the **Human Override** button.
4. **Step 4 (Reveal Hotspots & Action Brief)**:
   - Switch to the **"Hotspot Clusters"** tab.
   - Click **"View Action Brief"** on Hotspot #1 (Station Road).
   - Review the root-cause diagnosis: *"Blocked drains are washing market garbage into culverts, degrading road asphalt into craters."*
   - Click **"Print / Export PDF"** to preview the clean executive sign-off sheet.
5. **Step 5 (Close the Loop on Citizen Tracker)**:
   - Switch role to **"Citizen"** & open **"Citizen Tracker"**.
   - Enter your code (or pick from "My Recent Reports").
   - Notice the transparent plain-language message: *"Your report was prioritized and assigned to SWD Rapid Dewatering Gang #1. Work is actively in progress."*
6. **Reset**:
   - Click the circular arrow icon in the navbar to reset demo data back to the clean 60-report seed state at any time.

---

## 7. Limitations & Honesty Note (Mandatory Disclosure)

| Capability | Real | Simulated / Mocked | Explanation |
| :--- | :---: | :---: | :--- |
| **AI Classification & Deduplication** | **REAL** | — | Runs 100% locally via deterministic keyword weights and TF-IDF cosine vector math. |
| **Spatial Clustering** | **REAL** | — | Live DBSCAN-style distance grouping computes cluster boundaries and centroids. |
| **Explainable Factor Breakdown** | **REAL** | — | Mathematical point accounting prevents black-box distrust. |
| **Complaint Records** | — | **SIMULATED** | Pre-seeded with 60 synthetic reports; no real citizen PII is stored. |
| **Field Crews & Units** | — | **SIMULATED** | Dispatch assignments (e.g. BESCOM Gang #1) are fictional municipal teams. |
| **SMS / WhatsApp Alerts** | — | **SIMULATED** | Notifications are reflected in the in-app tracker rather than live telecom gateways. |
| **Authentication** | — | **SIMULATED** | Instant "Demo mode, no auth" role switcher for frictionless testing. |

### Where the AI Can Fail (Known Failure Modes):
1. **Slang / Non-Standard Dialects**: Heavy colloquial idioms without standard root keywords may produce lower confidence and fall back to the "Other" category.
2. **Mobile GPS Drift**: Urban canyons can introduce 50–100m GPS error, which could cause a report to fall just outside the 150m deduplication threshold.
3. **Photo Misalignment**: Without server-side image verification, a photo taken at a different location cannot be disproven without field officer validation.

### Next Steps for Production:
- On-premise fine-tuned lightweight SLM (Small Language Model) for sub-50ms municipal inference.
- Automated vision-based pothole depth and volume estimation from phone photos.
- Bidirectional integration with municipal GIS (ArcGIS / QGIS) asset infrastructure.

---

## 8. Deployment Steps (Netlify / Vercel)

ActionLens is client-side and requires zero database configuration to deploy:

### Deploy to Netlify
1. Connect your repository to Netlify.
2. Set Build Command: `npm run build`
3. Set Publish Directory: `dist`
4. (Optional) Set `VITE_LLM_API_KEY` in Netlify Environment Variables if Gemini enhancement is desired.

### Deploy to Vercel
1. Import repository into Vercel.
2. Framework Preset: **Vite**
3. Build Command: `npm run build`
4. Output Directory: `dist`
5. Click **Deploy**.
