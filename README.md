# KG-RAG: Knowledge Graph-based Retrieval Augmented Generation

This repository contains a collection of implementations for Knowledge Graph-based RAG (Retrieval Augmented Generation) approaches and baseline methods for comparison. The code is structured as a Python package with modular components.

## Overview

The repository implements several RAG approaches:

1. **Baseline approaches**:
   - **Standard RAG**: Traditional retrieval-based approach using vector similarity
   - **Chain-of-Thought RAG**: Enhanced retrieval with explicit reasoning steps

2. **KG-RAG approaches**:
   - **Entity-based approach**: Uses embedding-based entity matching and beam search to find relevant information in the knowledge graph
   - **Cypher-based approach**: Uses Cypher queries to retrieve information from a Neo4j graph database
   - **GraphRAG-based approach**: Implements a community detection and hierarchical search strategy

## Installation

### Using uv (Recommended)

This project uses [uv](https://github.com/astral-sh/uv) for dependency management.

```bash
# Clone the repository
git clone https://github.com/thillai-c/kg-rag.git
cd kg-rag

# Install uv if you don't have it
curl -sSf https://astral.sh/uv/install.sh | bash

uv sync
source .venv/bin/activate
```

**Note for Windows/PowerShell users**: Use `.venv\Scripts\Activate.ps1` instead of `source .venv/bin/activate`.

For development, you can install the dev dependencies:

```bash
uv sync --dev
source .venv/bin/activate
```

**Note for Windows/PowerShell users**: Use `.venv\Scripts\Activate.ps1` instead of `source .venv/bin/activate`.


## Environment Variables

Export the following environment variables:

```
OPENAI_API_KEY=your_openai_api_key
```

For the Cypher-based approach, also add:

```
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password
```

## Usage

### 1. Building Vector Store for Baseline Methods

First, build a vector store for the baseline RAG methods:

```bash
python -m scripts.build_baseline_vectordb \
    --docs-dir data/sec-10-q/docs \
    --collection-name sec_10q \
    --persist-dir chroma_db \
    --verbose
```

### 2. Building Knowledge Graphs

Build a knowledge graph for KG-RAG methods:

```bash
python -m scripts.build_entity_graph \
    --docs-dir data/sec-10-q/docs \
    --output-dir data/graphs \
    --graph-name sec10q_entity_graph \
    --verbose
```

### 3. Running Interactive Query Mode

To interactively query using baseline methods:

```bash
python -m scripts.run_baseline_rag \
    --collection-name sec_10q \
    --persist-dir chroma_db \
    --model gpt-4o \
    --verbose
```

To interactively query using KG-RAG methods:

```bash
python -m scripts.run_entity_rag \
    --graph-documents-pkl-path data/graphs/sec10q_entity_graph_documents.pkl \
    --top-k-nodes 10 \
    --top-k-chunks 100 \
    --similarity-threshold 0.5 \
    --verbose
```

### 4. Running Evaluation

To evaluate the performance of various RAG methods on a test dataset:

```bash
python -m kg_rag.evaluation.run_evaluation \
    --data-path data/sec-10-q/qna_data_mini.csv \
    --config-path kg_rag/configs/entity-based-kgrag.json \
    --method entity,baseline \
    --output-dir evaluation_results \
    --max-samples 50 \
    --verbose
```

**Note:** 
- The evaluation script uses configuration files (in `kg_rag/configs/`) to specify paths and parameters for each method. Make sure the config files point to the correct graph and document paths.
- You can use any of the CSV files in `data/sec-10-q/` as your test dataset.
- The script will automatically skip methods that require unavailable dependencies (e.g., Cypher method requires Neo4j).
- If your CSV uses different column names than "New Question" and "New Answer", use `--question-col` and `--answer-col` to specify them.

### 5. Running Hyperparameter Search

To find the optimal hyperparameters for a method:

```bash
python -m kg_rag.evaluation.hyperparameter_search \
    --data-path data/test_questions.csv \
    --graph-path data/graphs/sec10q_entity_graph.pkl \
    --method entity \
    --configs-path kg_rag/evaluation/hyperparameter_configs.json \
    --output-dir hyperparameter_search \
    --max-samples 10 \
    --verbose
```

## Development

### Pre-commit hooks

This project uses pre-commit hooks to ensure code quality:

```bash
# Run pre-commit hooks on all files
pre-commit run --all-files
```

### Running tests

```bash
# Run tests
pytest

# Run tests with coverage
pytest --cov=kg_rag tests/
```
