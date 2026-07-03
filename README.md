# IS Theory RAG Prototype

A minimal Retrieval-Augmented Generation (RAG) prototype that indexes **Information Systems theories** into a local vector database and answers natural-language queries with the most semantically relevant theories.

Everything runs locally in a single Jupyter notebook — no API keys, no external services. It uses:

- **[Weaviate](https://weaviate.io/) (embedded)** — an in-process vector database; no server to run.
- **[FastEmbed](https://github.com/qdrant/fastembed)** with the `BAAI/bge-small-en-v1.5` model — computes embeddings locally.

This is a learning prototype built while following the DeepLearning.AI × Astronomer course *Orchestrating Workflows for GenAI Applications* (see [Acknowledgments](#acknowledgments)).

## Prerequisites

- Python ≥ 3.12
- The repo ships with a checked-in `venv/`. If you prefer a fresh environment:
  ```bash
  python3 -m venv venv
  source venv/bin/activate
  pip install -r notebooks/requirements.txt
  ```

## Usage

Run from the `notebooks/` directory (paths in the notebook are relative to it):

```bash
source venv/bin/activate
cd notebooks
jupyter lab        # then open prototype.ipynb and run the cells top-to-bottom
```

The notebook, in order:

1. Starts an embedded Weaviate instance (data persists to `notebooks/tmp/weaviate/`).
2. Creates a `Theories` collection.
3. Parses every theory file in `notebooks/include/data/` into `{name, description, authors, seminal_articles}`.
4. Embeds each theory (name + description) and loads the vectors into Weaviate.
5. Runs a semantic query and prints the top matching theories.

Change the `query_str` variable in the query cell to search for different theories.

> **Note:** Embedded Weaviate binds ports `8079`/`50050`. If a previous run left a process behind, a new run fails with a "ports already listening" error — clear it with `pkill -f weaviate`.

### Data format

Each `notebooks/include/data/*.txt` file is **one theory** in Markdown. The `#` heading is the theory name; the `##` sections hold the content (any field may be `N/A`):

```
# <Theory name>
## Concise description of theory
<description or N/A>
## Originating author(s)
<authors or N/A>
## Seminal articles
<articles or N/A>
```

## License

The code in this repository is released under the [MIT License](LICENSE).

The MIT license covers **only the code**. The theory descriptions under `notebooks/include/data/` are third-party content (see [Acknowledgments](#acknowledgments)) and remain subject to their original terms.

## Acknowledgments

- **Course & structure** — This prototype follows the RAG walkthrough from the [*Orchestrating Workflows for GenAI Applications*](https://www.deeplearning.ai/courses/orchestrating-workflows-for-genai-applications) short course by [DeepLearning.AI](https://www.deeplearning.ai/) in partnership with [Astronomer](https://www.astronomer.io/), taught by Kenten Danas and Tamara Fingerlin. It is an independent reimplementation for learning purposes (the original example indexes book descriptions; this version indexes IS theories).
- **Theory data** — The theory descriptions were scraped from the [Information Systems Theories wiki](https://is.theorizeit.org/wiki/Main_Page) (`is.theorizeit.org`). Credit for that content belongs to the wiki and its contributors.
