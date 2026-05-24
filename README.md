# Analysis of Scientific Literature Using Graph Methods and ML

Bachelor thesis project at Innopolis University.

This repository contains the full data processing and analysis pipeline for a bibliometric study of chess research literature spanning from 1950 to the present day, developed as part of a bachelor thesis at Innopolis University.

The pipeline is designed as a **reusable methodology** — it is not specific to chess. Any collection of academic PDFs can be dropped into the `pdfs/` folder and the notebooks run top to bottom to produce a complete bibliometric analysis of that field: structured metadata, citation graphs, keyword co-occurrence networks, thematic clusters, and influence rankings.

The system works by extracting structured metadata from PDFs via GROBID, then enriching it with bibliographic data from the Crossref and OpenAlex APIs (DOIs, citation links, author names, publication years). From this enriched dataset it builds a heterogeneous knowledge graph connecting papers, authors, and keywords. Leiden community detection is applied to the knowledge graph to identify research communities and how topics evolved over time. Graph-theoretic influence metrics (PageRank, betweenness centrality, HITS authority scores, pioneer score) are computed to surface foundational papers and key authors across historical eras. All results are exported as interactive HTML graphs and CSV tables ready for inspection or further analysis.

## Pipeline order

Run the notebooks in the following order:

### 1. PDF preparation
| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/OCR.ipynb` | `pdfs/` | `pdfs_with_ocr/` — searchable PDFs |
| `scripts/PDF_to_XML.ipynb` | `pdfs_with_ocr/` | GROBID XML files |

### 2. Metadata extraction
| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/extract_XML.ipynb` | GROBID XML | `output/metadata_from_xml.csv` |

### 3. Metadata enrichment
| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/crossref.ipynb` | `output/metadata_from_xml.csv` | `output/metadata_enriched_FINAL.csv` |
| `scripts/citation.ipynb` | `output/metadata_enriched_FINAL.csv` | `output/metadata_enriched_after_openalex_*.csv`, `output/openalex_citations_cache.json` |

`crossref.ipynb` matches articles against the Crossref API (DOIs, authors, journals, references). `citation.ipynb` resolves remaining DOIs and fetches citation links via OpenAlex.

### 4. Keyword extraction
| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/llm_processing.ipynb` | `output/metadata_enriched_FINAL.csv`, PDFs | `keywords_results/` |

Extracts and normalises keywords using an LLM, then deduplicates and removes stopwords.

### 5. Graph construction & visualisation
| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/graph_vis.ipynb` | metadata + keywords | `output/articles_no_clusters.graphml/.html` |
| `scripts/cluster_graph_vis.ipynb` | metadata + keywords | `output/*_clusters.graphml/.html`, `output/cluster_report_*.csv` |
| `scripts/clusterization_dif_versions.ipynb` | metadata + keywords | `output/articles_{method}_clusters.graphml/.html` |

`graph_vis.ipynb` builds a plain keyword co-occurrence graph. `cluster_graph_vis.ipynb` applies Leiden clustering to keyword, author, and combined graphs. `clusterization_dif_versions.ipynb` compares seven clustering algorithms (Louvain, Leiden, Walktrap, Infomap, EM-SBM, Async Fluid, SCAN, Spinglass).

### 6. Knowledge graph metrics
| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/skg_metrix.ipynb` | metadata + keywords + citation cache | `output/graph/graph_metrics_enriched.csv`, `output/graph/author_metrics.csv`, `output/graph/cross_era_flow.csv`, `output/graph/keyword_emergence.csv` |

Builds the main heterogeneous knowledge graph (Paper / Author / Keyword), computes PageRank, betweenness, HITS, pioneer scores, and era-scoped metrics.

### 7. Research question analysis
| Notebook | Input | Output |
|----------|-------|--------|
| `scripts/graph_analysis.ipynb` | `output/graph/` | cluster CSVs, t-SNE plots, influence tables |
| `scripts/dbscan.ipynb` | embeddings | HDBSCAN cluster assignments |

`graph_analysis.ipynb` covers RQ2 (influential papers/authors across eras) and RQ3 (graph vs. clustering method comparison) using KMeans, Leiden, and HDBSCAN.

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
