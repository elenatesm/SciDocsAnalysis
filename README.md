# Analysis of Scientific Literature Using Graph Methods and ML

This repository contains the full data processing and analysis pipeline for a bibliometric study of chess research literature spanning from 1950 to the present day, developed as part of a bachelor thesis at Innopolis University.

The pipeline is designed as a **reusable methodology** — it is not specific to chess. Any collection of academic PDFs can be dropped into the `pdfs/` folder and the notebooks run in the pipeline order to produce a complete bibliometric analysis of that field: structured metadata, citation graphs, keyword co-occurrence networks, thematic clusters, and influence rankings.

The system works by extracting structured metadata from PDFs via GROBID, then enriching it with bibliographic data from the Crossref and OpenAlex APIs (DOIs, citation links, author names, publication years). From this enriched dataset it builds a heterogeneous knowledge graph connecting papers, authors, and keywords. Leiden community detection is applied to the knowledge graph to identify research communities and how topics evolved over time. Graph-theoretic influence metrics are computed to surface foundational papers and key authors across historical eras. All results are exported as interactive HTML graphs and CSV tables ready for inspection or further analysis.

# Pipeline Order

Run the notebooks in the following order:

---

## 1. PDF Preparation

| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/OCR.ipynb` | `pdfs/` | `pdfs_with_ocr/` — searchable PDFs |
| `scripts/PDF_to_XML.ipynb` | `pdfs_with_ocr/` | GROBID XML files |

---

## 2. Metadata Extraction

| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/extract_XML.ipynb` | GROBID XML | `output/metadata_from_xml.csv` |

---

## 3. Metadata Enrichment

| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/crossref.ipynb` | `output/metadata_from_xml.csv` | `output/metadata_enriched_FINAL.csv` |

`crossref.ipynb` matches articles against the Crossref API (DOIs, authors, journals, references).

---

## 4. Keyword Extraction

| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/llm_processing.ipynb` | `output/metadata_enriched_FINAL.csv`, PDFs | `keywords_results/` |

Extracts and normalises keywords using an LLM, then deduplicates and removes stopwords.

---

## 5. Embedding Clustering

| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/dbscan.ipynb` | embeddings | HDBSCAN cluster assignments |

Clusters papers by embedding similarity using HDBSCAN. Run before graph construction so that cluster assignments are available to all downstream keyword-based graphs.

---

## 6. Citation & Graph Construction

| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/citation.ipynb` | `output/metadata_enriched_FINAL.csv` | `output/metadata_enriched_after_openalex_*.csv`, `output/openalex_citations_cache.json` |
| `scripts/graph_vis.ipynb` | metadata + keywords | `output/articles_no_clusters.graphml/.html` |
| `scripts/cluster_graph_vis.ipynb` | metadata + keywords | `output/*_clusters.graphml/.html`, `output/cluster_report_*.csv` |

`citation.ipynb` resolves remaining DOIs and fetches citation links via OpenAlex, producing the citation edge graph used in downstream visualisations. `graph_vis.ipynb` builds a plain keyword co-occurrence graph. `cluster_graph_vis.ipynb` applies Leiden clustering to keyword, author, and combined graphs.

---

## 7. Knowledge Graph Metrics

| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/skg_metrix.ipynb` | metadata + keywords + citation cache | `output/graph/graph_metrics_enriched.csv`, `output/graph/author_metrics.csv`, `output/graph/cross_era_flow.csv`, `output/graph/keyword_emergence.csv` |

Builds the main heterogeneous knowledge graph (Paper / Author / Keyword), computes PageRank, betweenness, HITS, pioneer scores, and era-scoped metrics.

---

## 8. Research Question Analysis

| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/graph_analysis.ipynb` | `output/graph/` | cluster CSVs, t-SNE plots, influence tables |

Covers RQ2 (influential papers/authors across eras) and RQ3 (graph vs. clustering method comparison) using KMeans, Leiden, and HDBSCAN.

## Output directory structure

```
scripts/                — all Jupyter notebooks
results/
├── graphs/             — interactive pyvis HTML graphs of the knowledge graph and citation network
└── graph_analytics/    — analysis plots: cluster profiles, t-SNE projections, influence charts, timeline visualisations
```

## Requirements

```
pip install -r requirements.txt
```

GROBID must be running locally for `PDF_to_XML.ipynb` (see [GROBID docs](https://grobid.readthedocs.io/en/latest/Grobid-service/)).
