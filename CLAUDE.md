# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A minimal RAG (Retrieval-Augmented Generation) prototype that indexes **Information Systems theories** into a vector database and answers recommendation queries via semantic search. The entire pipeline lives in a single Jupyter notebook (`notebooks/prototype.ipynb`); there is no application entry point or test suite. It is built around a DeepLearning.AI-style walkthrough (originally a book recommender, repurposed for IS theories).

## Environment & Commands

The project uses a checked-out `venv/` (Python ≥ 3.12) and must be run from the `notebooks/` directory, since paths in the notebook are relative to it.

```bash
source venv/bin/activate
pip install -r notebooks/requirements.txt   # weaviate-client==4.14.1, fastembed==0.6.1, ipython
cd notebooks && jupyter lab                  # then run prototype.ipynb top-to-bottom
```

The notebook is stateful and cells are meant to run in order. The last two (commented-out) cells hold cleanup utilities — dropping the collection and deleting the embedded DB — for resetting state. Embedded Weaviate binds ports `8079`/`50050`; if a run leaves a `weaviate` process behind, a fresh run fails with a "ports already listening" error — `pkill -f weaviate` clears it.

## Architecture

The pipeline (each stage is a notebook section):

1. **Embedded Weaviate** — `weaviate.connect_to_embedded(persistence_data_path="tmp/weaviate")` spins up an in-process vector DB. There is no external server; data persists to `notebooks/tmp/weaviate/`. Collection name is `Theories` (Weaviate requires a capitalized first letter).
2. **Local embedding model** — `fastembed.TextEmbedding("BAAI/bge-small-en-v1.5")`. Embeddings are computed manually in Python; the Weaviate collection is created without a vectorizer, and vectors are supplied explicitly on insert (`DataObject(properties=..., vector=emb)`) and on query (`near_vector`).
3. **Ingestion** — reads every `*.txt` file in `notebooks/include/data/` (one theory per file) via `parse_theory_file()`, producing a flat `list_of_theory_data` of `{name, description, authors, seminal_articles}` dicts.
4. **Embedding** — embeds `f"{name}. {description}"` per theory so the name adds signal. For the ~16 theories whose description is `N/A`, only the **name** is embedded (so they stay retrievable). `authors` and `seminal_articles` are stored as properties but deliberately **not** embedded — they're citations/proper nouns that would add noise to conceptual similarity.
5. **Query** — embeds a query string and runs `collection.query.near_vector(..., limit=3)` to return the closest theories.

### Theory file format

Each `include/data/*.txt` file is **one theory** in Markdown. The `#` H1 is the theory name; three `##` sections carry the content; any field may be `N/A` (normalized to an empty string during parsing):

```
# <Theory name>
## Concise description of theory
<description or N/A>
## Originating author(s)
<authors or N/A>
## Seminal articles
<articles or N/A>
```

The source corpus lives in a sibling repo, `../0-scraper/scraped-text-files/`; the 158 files were copied into `include/data/`. To refresh the corpus, re-copy from the scraper output.

## Important: local-only artifacts

`.gitignore` excludes `models/`, `venv/`, `notebooks/tmp/` (the persisted Weaviate DB), and the byte-compiled/`__pycache__` dirs. All of these exist in the working tree but are **not** tracked — none of them (in particular the `bge-small-en-v1.5` model weights) are in git history. The `bge-small-en-v1.5` model must exist locally for the notebook to run offline; download it into `models/` (e.g. via `huggingface-cli`) if it is missing.

## Utilities

`notebooks/helper.py` provides `suppress_output()` — a context manager that redirects stdout/stderr at the file-descriptor level (so it silences noisy native child processes like the embedded Weaviate startup logs, not just Python prints).
