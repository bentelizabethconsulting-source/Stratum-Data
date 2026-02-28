# Stratum — Cross-Modal Data Intelligence Platform

**A design prototype and open invitation to build.**

Stratum is a concept for a data analysis platform that takes large text corpora — emails, documents, messages — extracts every keyword from the body text, builds a document-term matrix, discovers co-occurring keyword modules via network analysis, and then uses the timestamps on those modules as a join key into structured numeric datasets (bank transfers, flight logs, purchases, access records) to find cross-modal co-occurrences that neither dataset could reveal alone.

This repository currently contains a **front-end design prototype** (`index.html`) — a fully rendered website describing the platform's intended capabilities. There is no working backend yet. The purpose of making this public is to find collaborators who can validate the analytical approach, identify where the methodology is sound or flawed, and help build toward a real implementation.

---

## What the platform proposes to do

### Phase 1 — Text corpus analysis

1. Ingest a text corpus (emails, documents, records) via API or file upload
2. Extract all keywords from body text using NLP preprocessing: tokenization, stopword removal, lemmatization, optional n-gram generation
3. Filter vocabulary by document frequency thresholds to control matrix size
4. Build a **document-term matrix** (rows = documents, columns = extracted keywords, cells = TF-IDF weights)
5. Compute column-wise correlations to produce a **term co-occurrence network**
6. Run community detection (Louvain) to identify **keyword modules** — clusters of terms that co-occur significantly across the corpus without being pre-defined by the analyst
7. Validate discovered modules using Structural Equation Modeling (SEM) — treating each module's keyword set as indicators of a latent variable

### Phase 2 — Cross-modal temporal fusion

8. Extract the timestamps of documents belonging to each keyword module, creating a **module activity time series**
9. Accept a second, structured dataset (e.g. bank transfers, flight logs, purchases) that also has timestamps
10. Fuse the two datasets by time using five analytical modes:
    - **Temporal Overlay** — visual co-occurrence of module activity and event frequency
    - **Lagged Cross-Correlation** — does text activity lead or follow structured events, and by how long?
    - **Event-Driven Windowing** — what structured events cluster inside keyword module activity windows?
    - **Shared Entity Linkage** — named entities appearing in both datasets as direct semantic bridges
    - **Joint SEM** — both modalities as indicators of the same latent construct

---

## What exists right now

| Component | Status |
|---|---|
| Website / design prototype (`index.html`) | ✅ Complete |
| NLP preprocessing pipeline | ❌ Not built |
| Document-term matrix construction | ❌ Not built |
| Term co-occurrence network computation | ❌ Not built |
| Louvain community detection | ❌ Not built |
| SEM implementation | ❌ Not built |
| Module activity time series extraction | ❌ Not built |
| Cross-modal temporal fusion engine | ❌ Not built |
| Visualizations (network graph, timeline, heatmap) | ❌ Not built (mocked in HTML) |
| API connectors (email, database, REST) | ❌ Not built |

---

## Where we need help

This project needs people who can evaluate the methodology, identify problems with it, and contribute to building it. Specific areas:

### Statisticians / Psychometricians
The SEM component is the most methodologically complex and the most likely to be implemented naively. Key open questions:

- Is using network-detected keyword modules as CFA indicator sets for latent variables methodologically sound, or does it introduce circularity?
- What sample size (number of documents) is required for stable SEM estimation at this vocabulary scale?
- Which fit indices are most appropriate given the sparse, non-normal nature of TF-IDF-weighted data?
- Should the SEM be estimated on the raw DTM, a reduced representation (e.g. PCA scores), or the module membership scores?
- Is a WGCNA-style (weighted gene co-expression network analysis) approach more appropriate than Louvain for this data structure?

### NLP / Computational Linguists
- What preprocessing choices matter most for downstream network structure (stemming vs. lemmatization, bigram inclusion thresholds, TF-IDF variant)?
- At what vocabulary size does the co-occurrence network become too dense to yield meaningful modules?
- Are there better alternatives to TF-IDF weighting for this use case (BM25, PMI, PPMI)?
- How should named entity recognition be integrated to support the Shared Entity Linkage fusion mode?

### Data Engineers / Backend Developers
- The DTM at 10M documents × 100K terms is too large for dense representation — what sparse matrix stack is most appropriate (scipy.sparse, Polars, DuckDB)?
- What network library is best suited for this scale (NetworkX, igraph, graph-tool)?
- Architecture for the cross-modal join: how should the temporal windowing be implemented efficiently when one dataset has millions of timestamped rows?

### Domain Experts
If you work in forensic accounting, fraud detection, intelligence analysis, clinical informatics, or computational social science — these are the fields where cross-modal temporal fusion of text and structured records is most likely to have real-world value. We want to know: does the approach make sense in your domain, and what would a meaningful test dataset look like?

---

## Suggested tech stack (open to challenge)

| Layer | Proposed | Alternatives welcome |
|---|---|---|
| NLP preprocessing | spaCy | NLTK, Stanza |
| DTM construction | scikit-learn `TfidfVectorizer` | Gensim, custom |
| Sparse matrix ops | scipy.sparse | Polars, DuckDB |
| Network analysis | igraph + leidenalg | NetworkX, graph-tool |
| SEM | semopy (Python) | lavaan (R), pySEM |
| Dimensionality reduction | UMAP + scikit-learn | Other |
| Time series / cross-correlation | statsmodels | tsfresh, other |
| Frontend | Vanilla HTML/CSS/JS | Open to React |
| Backend API | FastAPI | Flask, other |

---

## How to run the prototype

The current prototype is a single static HTML file. No build step, no dependencies.

```bash
git clone https://github.com/YOUR_USERNAME/Stratum-Data.git
cd stratum
open index.html   # or just double-click the file
```

That's it. Everything you see is mocked — the matrix values, network graph, and timeline visualizations are all static. They are illustrations of intended output, not computed results.

---

## Contributing

All contributions are welcome. The most valuable ones right now are **critiques** — if the SEM approach is flawed, if the network analysis methodology has a known problem at this scale, or if there is a better-established technique for any part of this pipeline, please open an Issue and explain it.

To contribute:

1. Fork the repository
2. Create a branch (`git checkout -b feature/your-contribution`)
3. Commit your changes
4. Open a Pull Request with a description of what you've added or changed and why

For methodological discussions, open an **Issue** rather than a PR. Tag it with the relevant label (`methodology`, `nlp`, `sem`, `fusion`, `architecture`).

---

## Intended use cases

The platform is designed for analysts working with datasets where the relationship between text communication and real-world events is the object of study. Example domains:

- **Investigative / forensic** — correlating communication records with financial transactions or travel records
- **Organizational research** — understanding how language in internal communications relates to operational decisions
- **Clinical informatics** — fusing clinical notes keyword modules with coded encounter or prescription data
- **Computational social science** — correlating text corpus activity (news, social media, legislative records) with economic or political event data

---

## License

MIT License. See `LICENSE` for details.

---

## Contact

Open an Issue on this repository. All design and methodology questions are in scope.
