# Stratum-Data — Design Notes
*Conversation log and methodology rationale. Add this file to the Stratum-Data GitHub repository so future contributors can understand the reasoning behind the design.*

---

## Origin of the concept

The platform was conceived around a specific use case: taking a large email corpus (used as the example: JMail.world) and instead of searching it with individual keywords or small sets of terms, analysing the full body text of every email simultaneously to find patterns that no one knew to search for.

The core insight is that **keyword search is hypothesis-driven** — you find what you already suspect is there. The goal of Stratum is **hypothesis-free discovery** — finding structure in the data that emerges from the mathematics rather than from prior assumptions.

---

## Key clarification made during design

An important correction was made early in the design process:

> The columns in the matrix are **keywords extracted from the body text of the emails**, not pre-defined annotation categories or subject line labels.

This is a critical distinction. The original prototype incorrectly showed columns like "Finance", "Legal", "Project" — as if a human had pre-labelled those dimensions. The correct model is:

- **Rows** = individual emails (documents)
- **Columns** = keywords extracted from email body text via NLP
- **Cells** = TF-IDF weight of that keyword in that email

This is a standard **document-term matrix (DTM)**, also called a term-document matrix. The novelty is not the matrix itself but what is done with it: network analysis and SEM rather than just search or topic modelling.

---

## Phase 1 — Text corpus analysis pipeline

### Step 1: Ingest
- Connect to email system (JMail.world, Gmail, Outlook, IMAP, etc.)
- Pull **full body text** of every message — not just subject lines or metadata
- Retain sender, recipient, timestamp, thread ID as document-level attributes

### Step 2: Keyword extraction (NLP preprocessing)
- Tokenisation
- Stopword removal
- Lemmatisation (preferred over stemming for interpretability)
- Optional n-gram generation (e.g. "cash flow", "project deadline" as single tokens)
- Frequency thresholding — remove terms that appear in fewer than N documents (noise) or more than X% of documents (too ubiquitous to be informative)
- Resulting vocabulary for a corpus of ~5,000 emails: typically 10,000–50,000 terms after filtering

### Step 3: Build the document-term matrix
- Sparse matrix: most cells are zero (most words don't appear in most emails)
- Cell weighting options: binary presence, raw count, TF-IDF (recommended), BM25
- TF-IDF logic: high weight for terms that are distinctive to specific emails; low weight for terms that appear everywhere
- Store as sparse matrix (scipy.sparse or equivalent) — a dense matrix at 5,000 × 20,000 is 100M cells, manageable; at 10M × 100K it is not

### Step 4: Compute the term co-occurrence network
- Compute **column-wise correlations** across the DTM
- This produces a term × term co-occurrence matrix (e.g. 18,340 × 18,340)
- Each cell = how often two keywords appear together across emails
- Strong correlations become edges in a network graph
- Nodes = keywords; edges = co-occurrence strength

### Step 5: Community detection — keyword modules
- Run Louvain community detection (or Leiden algorithm) on the term co-occurrence network
- Output: **keyword modules** — clusters of terms that co-occur significantly without being pre-defined
- Example output from JMail.world prototype:
  - **Module A** — "budget", "liability", "exposure", "audit" → Financial/Legal cluster (31% of corpus)
  - **Module B** — "milestone", "deliverable", "overdue", "deadline" → Project/Ops cluster (44% of corpus)
  - **Module C** — "compliance", "urgent", "escalate" → Compliance/Urgency cluster (18% of corpus)
- These modules were not defined in advance — they emerged from the mathematics

### Step 6: Validate with Structural Equation Modeling (SEM)
- Treat each keyword module's term set as **indicators of a latent variable**
- Run Confirmatory Factor Analysis (CFA) to test whether the terms load coherently onto a single factor
- Fit indices to report: CFI (>0.90 acceptable, >0.95 good), RMSEA (<0.08 acceptable, <0.05 good), SRMR
- A good fit confirms the module reflects a real latent construct, not statistical noise
- Example fit from prototype: CFI = 0.94, RMSEA = 0.048 → three-factor structure confirmed

### Open methodological questions (SEM)
These were explicitly flagged as needing expert validation:
1. Is using network-detected modules as CFA indicator sets circular? (The same data used to find the modules is used to validate them)
2. What minimum document count is needed for stable SEM estimation at this vocabulary scale?
3. TF-IDF data is non-normal and sparse — which estimator is appropriate? (MLR? WLSMV?)
4. Should SEM be run on raw DTM values, PCA-reduced scores, or module membership scores?
5. Is WGCNA (weighted gene co-expression network analysis, from genomics) a better-established method for this data structure than Louvain?

---

## Phase 2 — Cross-modal temporal fusion

### The core idea
Each keyword module has a **temporal signature**: the distribution of timestamps from emails where those terms appear. This becomes a **module activity time series** — a signal of when the corpus was discussing that theme.

Any other dataset that also has timestamps can then be searched for patterns that co-occur with, precede, or follow spikes in module activity.

**Time is the join key** between otherwise incomparable datasets.

### Datasets that can be fused
- Bank transfers (amount, counterparty, timestamp)
- Flight logs (route, departure time, tail number)
- Purchase records (merchant, category, amount, timestamp)
- Access logs (building entry, system logins, badge scans)
- Call detail records / CDRs (caller, recipient, duration, timestamp)
- Shipping and logistics records
- Clinical encounter data (diagnoses, procedures, prescriptions, dates)
- Any table with a datetime column and at least one other field

### The five fusion modes

**Mode 1 — Temporal Overlay**
Plot module activity time series alongside event frequency from the structured dataset on the same time axis. Visual alignment reveals whether spikes co-occur. No statistical claims made — purely exploratory.

**Mode 2 — Lagged Cross-Correlation**
Compute cross-correlation at multiple time lags (e.g. −7 days to +7 days). Identifies whether text activity *predicts* structured events, *follows* them, or merely coincides — and by how many hours or days.
- Example: Module A email activity precedes anomalous wire transfers by ~4 hours (r = 0.74, lag = +4h)

**Mode 3 — Event-Driven Windowing**
For each email tagged to a module, open a configurable time window (e.g. ±48 hours) and extract all structured records within it. Build a secondary co-occurrence matrix: which event types cluster inside Module A windows vs. Module B windows?
- Example: Module C emails are followed within 6 hours by international flight bookings in 38% of cases

**Mode 4 — Shared Entity Linkage**
When the structured dataset contains named entities — merchant names, airline codes, account numbers — that also appear as extracted keywords in the email corpus, those records become **direct semantic bridges** between the two datasets. The link is both temporal and lexical.
- Example: "Acme Corp" appears in Module A emails and as a transfer recipient on the same dates

**Mode 5 — Joint SEM**
Build a Structural Equation Model where both keyword module loadings (from text) and structured event patterns (from numeric data) are indicators of the same latent construct. Tests whether the two modalities jointly reflect a single underlying process.
- Example: A latent "financial stress" factor loads on Module A keywords AND on wire transfer frequency and amounts

### Bonus mode — Anomaly Alerting
Once a baseline co-occurrence pattern is established between a keyword module and a structured event type, flag deviations: module spikes with no corresponding structured event in the expected window, or unusual events arriving outside typical module activity periods.

---

## Design and front-end decisions

### Technology
- Single-file static HTML prototype (`index.html`) — no build step, no dependencies, opens in any browser
- Fonts: Calibri (system font, no load required), Syne (display headings), Instrument Serif (italic accents)
- Colour scheme: dark background (#05080f), bright green accent (#00e5a0), purple (#7b6ff0), amber (#f0a040)
- Muted/secondary text: light green (#90e8b0) — changed from grey on user request for readability

### Visualisations in the prototype (all mocked, not computed)
- Document-term matrix table with TF-IDF cell values and module colour coding
- Keyword co-occurrence network graph (SVG, three modules colour-coded)
- Module activity vs. structured event timeline with co-occurrence window overlay
- Join key diagram showing email corpus and bank transfer log side by side

---

## Repository structure

```
Stratum-Data/
├── index.html       # Front-end design prototype (complete)
├── README.md        # Repository description and contributor guide
├── NOTES.md         # This file — design rationale and conversation record
└── LICENSE          # MIT licence (to be added)
```

---

## Suggested backend tech stack

| Layer | Proposed | Notes |
|---|---|---|
| NLP preprocessing | spaCy | NLTK or Stanza are alternatives |
| DTM construction | scikit-learn TfidfVectorizer | Gensim for larger corpora |
| Sparse matrix ops | scipy.sparse | Polars or DuckDB for very large scale |
| Network analysis | igraph + leidenalg | NetworkX simpler but slower at scale |
| SEM | semopy (Python) | lavaan (R) more mature and better validated |
| Dimensionality reduction | UMAP + scikit-learn PCA | |
| Time series / cross-correlation | statsmodels | tsfresh for feature extraction |
| Backend API | FastAPI | Flask if simpler preferred |
| Frontend (future) | Vanilla JS or React | Current prototype is vanilla HTML |

---

## Intended use cases

- **Investigative / forensic analysis** — correlating communication records with financial transactions, travel records, or access logs without knowing in advance what to look for
- **Organisational research** — understanding how language in internal communications relates to operational decisions and events
- **Clinical informatics** — fusing clinical notes keyword modules with coded encounter, prescription, or lab data
- **Computational social science** — correlating text corpus activity (news, social media, legislative records) with economic or political event data
- **Fraud detection** — identifying co-occurrence patterns between communication themes and transaction anomalies

---

## What to build first

Suggested order for a backend implementation:

1. **DTM pipeline** — get text in, get a sparse TF-IDF matrix out. This is well-trodden ground with scikit-learn and is the foundation everything else depends on.
2. **Network computation** — column-wise correlation + Louvain/Leiden. igraph makes this straightforward.
3. **Module timestamp extraction** — for each module, collect the timestamps of documents with high module membership. This is simple once modules exist.
4. **Temporal overlay visualisation** — plot module activity alongside a structured dataset. Validates the fusion concept before building the full cross-correlation machinery.
5. **Lagged cross-correlation** — statsmodels ccf() is a one-liner once the time series exist.
6. **SEM validation** — do this last and with expert input. semopy in Python or lavaan in R. Don't implement this without statistical review of the methodology questions listed above.

---

*Notes compiled from design conversation, February 2026.*
