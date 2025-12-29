
# Genome Assembly & Functional Annotation

## 🧬 Project Overview

This repository documents the computational workflow for **De Novo Genome Assembly**. Unlike re-sequencing, which relies on a scaffold, de novo assembly is an exercise in **inductive logic**: we take millions of stochastic DNA fragments (reads) and infer the underlying biological architecture of an organism's entire genetic code.

### The Significance of Assembly

Genome assembly is the "ground truth" of biology. It is essential for:

* **Phylogenetic Inference:** Understanding the evolutionary trajectory between species.
* **Metagenomics:** Deconvoluting complex microbial communities in environmental or clinical samples.
* **Structural Variant Discovery:** Identifying large-scale genomic rearrangements that standard mapping often misses.

---

## ⚙️ The Technical Mechanics: From Reads to Contigs

The primary challenge of assembly is the **Short Read Problem**. Because genomes contain repetitive regions, we cannot simply "stitch" reads together linearly. Instead, we use **De Bruijn Graphs**.

### 1. K-mer Decomposition

We break every read into overlapping strings of length , known as **k-mers**.

* **The Logic:** If , two reads that share a 30-base overlap will converge at a specific node in our graph.
* **The Trade-off:** A small  increases sensitivity but introduces **ambiguity** in repetitive regions; a large  increases specificity but requires higher sequencing depth.

### 2. Eulerian Paths

The assembler treats the genome as a directed graph. The goal is to find an **Eulerian path**—a trail that visits every edge in the graph exactly once. This reduces the assembly problem from a massive string-matching task to a solved problem in topology.

---

## 🛠 Workflow & Methodology

### Phase I: Quality Control (QC)

Before assembly, we must apply **deductive filtering** to remove low-quality data.

* **Metric:** We use the **Phred Quality Score** ().



*(Where  is the probability of an incorrect base call. A  score indicates a 99.9% accuracy rate.)*

### Phase II: Assembly Pipeline

1. **Read Trimming:** Removing adapter sequences and low-Q bases using `Trimmomatic`.
2. **Assembly:** Invoking `SPAdes` or `SKESA` to construct the De Bruijn graph and output **contigs** (contiguous sequences).
3. **Scaffolding:** Using paired-end information to orient contigs and fill gaps with `N`s.

### Phase III: Functional Annotation

Once the sequence is assembled, we transition from **syntax** (the code) to **semantics** (the meaning).

* **Gene Prediction:** Using Hidden Markov Models (HMMs) in tools like `Prokka` to identify Open Reading Frames (ORFs).
* **Homology Search:** Comparing predicted proteins against the UniProt or RefSeq databases using `BLAST` to assign biological functions.

---

## 📊 Evaluation Metrics

We don't just "finish" an assembly; we validate it through **statistical reduction**:

* **N50 Statistic:** A measure of assembly "contiguity." It is the length of the shortest contig such that all contigs of that length or longer contain 50% of the total assembly.
* **BUSCO:** Benchmarking Universal Single-Copy Orthologs. This tells us how "complete" the genome is based on expected evolutionary conserved genes.

---

## 🚀 Getting Started for Mentees

1. **Clone the Repo:** `git clone https://github.com/your-username/Genome-Assembly-Annotation.git`
2. **Environment Setup:** We use `Conda` to ensure **computational reproducibility**.
`conda env create -f environment.yml`
3. **Execute the Pipeline:**
`bash scripts/run_assembly.sh`

