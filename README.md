# Implementation of A MVC Quantum Decomposition Algorithm

This repository contains the code and dissertation material for a thesis on decomposition-based approaches to NP-hard graph problems for quantum annealing. The main focus is the implementation and evaluation of a decomposition algorithm for the Minimum Vertex Cover problem, with a companion implementation for Maximum Clique.

The project builds on the decomposition ideas introduced in the DASQA work by Pelofske, Hahn, and Djidjev, and adapts them into a thesis-oriented codebase with exact validation scripts and D-Wave-based solver backends.

## Thesis document

The dissertation is included in this repository as `thesis-dissertation.pdf`.

- Title: `Implementation of A MVC Quantum Decomposition Algorithm`
- Author: `Konstantinos Larkou`

## What this repository contains

- `DBR.py`: implementation of the decomposition algorithm for Minimum Vertex Cover.
- `validate_DBR.py`: experiment script that compares the decomposition pipeline against an exact classical baseline and D-Wave-backed solvers.
- `MVC_chimera.py`: Minimum Vertex Cover solver wrapper targeting a D-Wave Chimera topology.
- `MVC_pegasus.py`: Minimum Vertex Cover solver wrapper targeting a D-Wave Pegasus topology.
- `maximum_clique/DBK.py`: decomposition algorithm for Maximum Clique.
- `maximum_clique/MC_CQM.py`: D-Wave CQM-based Maximum Clique solver.
- `maximum_clique/validate_DBK.py`: validation script for the Maximum Clique pipeline.
- `maximum_clique/mc_test.py`: small standalone Maximum Clique test script.
- `maximum_clique/run.sh`: remote execution helper used for experiments.

## Project idea

Quantum annealers cannot solve arbitrarily large graph instances directly, so the core idea is to decompose a large graph into smaller subproblems that fit a chosen solver limit.

- `DBR` recursively partitions a graph for Minimum Vertex Cover, applies reduction rules, keeps track of forced vertices, and sends only sufficiently small subgraphs to a solver function.
- `DBK` does the analogous job for Maximum Clique, using bounds and reductions to prune the search space before solving subgraphs.
- The solver is injected as a function argument, so the decomposition layer can be paired with either exact classical solvers or D-Wave samplers.

## Requirements

The repository does not include a `requirements.txt`, so dependencies need to be installed manually.

Core Python packages used in the code:

- `networkx`
- `numpy`
- `matplotlib`
- `jgrapht`
- `dimod`
- `dwave-system`

Recommended environment:

- Python 3.10+
- Access to a D-Wave Leap account if you want to run the quantum solver scripts

Example installation:

```bash
pip install networkx numpy matplotlib python-jgrapht dimod dwave-system
```

To run the D-Wave-backed scripts, configure your credentials through the standard D-Wave tooling, for example with the `DWAVE_API_TOKEN` environment variable or a local D-Wave config file.

## How to use

### 1. Use the Minimum Vertex Cover decomposition directly

`DBR` expects:

- a `networkx.Graph`
- an integer `LIMIT`
- a solver function that accepts a graph and returns a vertex cover for that graph

Minimal example with an exact classical solver:

```python
import networkx as nx
from DBR import DBR

def exact_mvc_solver(G):
    gc = nx.complement(G)
    max_clique_size = nx.graph_clique_number(gc)
    for clique in nx.find_cliques(gc):
        if len(clique) == max_clique_size:
            return list(set(G.nodes()) - set(clique))

G = nx.gnp_random_graph(40, 0.2, seed=7)
solution = DBR(G, LIMIT=15, solver_function=exact_mvc_solver)
print(solution)
```

### 2. Run the Minimum Vertex Cover experiment script

```bash
python validate_DBR.py
```

This script:

- generates random graphs
- runs the original decomposition method
- runs Pegasus and Chimera solver backends
- compares the returned cover sizes and execution times
- saves graph visualizations as `.png` files

### 3. Run the Maximum Clique code

The Maximum Clique implementation lives in `maximum_clique/`.

Main files:

- `maximum_clique/DBK.py`
- `maximum_clique/MC_CQM.py`
- `maximum_clique/validate_DBK.py`

Depending on your environment, some of the research scripts may need small path or import adjustments before they run unchanged.

## Notes on the codebase

- This is research code, not a packaged Python library.
- Several scripts are written as experiment drivers rather than reusable CLIs.
- Output files such as timing logs and generated figures are written into the working directory.
- The D-Wave solvers assume access to compatible hardware and credentials.

## Research background

This repository is closely related to the decomposition algorithms introduced in:

- Pelofske, Hahn, and Djidjev, `Solving large maximum clique problems on a quantum annealer`, QTOP 2019.
- Pelofske, Hahn, and Djidjev, `Solving large minimum vertex cover problems on a quantum annealer`, Computing Frontiers 2019.
- Pelofske, Hahn, and Djidjev, `Decomposition algorithms for solving NP-hard problems on a quantum annealer`, Journal of Signal Processing Systems, 2021.

Upstream project:

- `lanl/Decomposition-Algorithms-for-Scalable-Quantum-Annealing`

## Attribution

The repository appears to be based on the original LANL DASQA codebase and adapted for thesis work. If you use this repository in academic work, cite both the dissertation and the upstream research papers that introduced the decomposition methods.
