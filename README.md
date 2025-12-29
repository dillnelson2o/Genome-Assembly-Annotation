# Genome Reconstruction (ONT)

## 🧬 Project Overview
This repository documents the computational workflow for **De Novo Genome Assembly** using Oxford Nanopore Technologies (ONT). In this project, we move beyond "re-sequencing" (mapping to a known template) and instead perform an exercise in **inductive logic**: we use the raw physical overlaps of stochastic DNA fragments to infer the original biological architecture of a genome.

### The Significance of Long-Read Assembly
Genome assembly is the "ground truth" of biology. Using ONT long reads allows us to solve the **Short Read Problem** by spanning complex repetitive regions that are otherwise computationally **intractable**. 

* **Phylogenetic Inference:** Understanding evolutionary trajectories with high-resolution genomic data.
* **Structural Variant Discovery:** Identifying large-scale rearrangements (insertions, inversions) that short reads often miss.
* **Functional Annotation:** Transitioning from "syntax" (nucleotide strings) to "semantics" (gene function).

---

## ⚙️ The Technical Mechanics: Overlap-Layout-Consensus (OLC)
Unlike short-read assemblers that use De Bruijn Graphs, long-read assembly typically utilizes the **OLC paradigm**. 

1.  **Overlap:** Every read is compared against others to find shared sequences.
2.  **Layout:** A graph is constructed where reads are nodes and overlaps are edges. We simplify this graph to find the most likely linear path (a Hamiltonian path).
3.  **Consensus:** Because raw ONT reads have a higher error rate, we use statistical "voting" and Bayesian polishing to determine the correct base calls.



---

## 🛠 The Computational Pipeline
The workflow is segmented into four logical phases, executed via modular environments on the **Innovator HPC**:

| Phase | Methodology | Goal |
| :--- | :--- | :--- |
| **I. QC & Filtering** | Deductive pruning of "adapter" noise and artifacts. | Ensure data integrity. |
| **II. OLC Assembly** | Constructing an overlap graph via `Flye`. | Maximizing **N50** (Contiguity). |
| **III. Polishing** | Neural Network-based error correction via `Medaka`. | Maximizing **Q-Score** (Accuracy). |
| **IV. Annotation** | HMM-based gene prediction via `Prokka`. | Functional Interpretation. |

