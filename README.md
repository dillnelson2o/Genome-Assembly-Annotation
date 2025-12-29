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

---
# 🧬 De Novo Genome Reconstruction & Annotation (ONT)

This repository documents the computational workflow for **De Novo Genome Assembly** using Oxford Nanopore Technologies (ONT) long-read sequencing. This project was developed as a pedagogical framework for high school students to explore bioinformatics through the lens of **graph theory** and **inductive logic**.

---

## 🔬 Project Philosophy

### The Logic of Long Reads
Unlike traditional short-read sequencing, which functions like a fragmented puzzle, ONT provides long reads that allow us to span complex genomic repeats. 

1.  **Inductive Assembly:** We do not rely on a reference "map." Instead, we use the raw physical overlaps of DNA molecules to infer the underlying biological architecture.
2.  **The Noise-Length Trade-off:** While ONT provides massive **contiguity** (long fragments), it introduces stochastic base-calling errors. Our workflow applies **Bayesian polishing** to refine the final consensus sequence.
3.  **Biological Semantics:** We move from **syntax** (the raw $A, T, C, G$ code) to **semantics** (the meaning of genes) through functional annotation.

---

## ⚙️ The Technical Mechanics: Overlap-Layout-Consensus (OLC)
In this project, we utilize the **OLC paradigm** to reconstruct the genome:

* **Overlap:** Identifying shared sequences between reads via all-pairs alignment.
* **Layout:** Simplifying the overlap graph to find the most probable linear path.
* **Consensus:** Using neural networks to "vote" on the correct nucleotide at every position.

[Image of Overlap-Layout-Consensus OLC assembly algorithm]

---

## 🛠 Workflow & Methodology
The analysis is performed on the **Innovator HPC (SDState)** using modular environments to ensure computational reproducibility.

| Phase | Methodology | Tools | Goal |
| :--- | :--- | :--- | :--- |
| **I. Quality Control** | Deductive filtering | `NanoPlot`, `FastQC` | Ensure data integrity and $Q$-score validity. |
| **II. Assembly** | OLC Reconstruction | `Flye`, `Minimap2` | Maximize **N50** (Contiguity). |
| **III. Polishing** | Neural Network Correction | `Medaka`, `Racon` | Maximize Consensus Accuracy. |
| **IV. Annotation** | HMM Gene Prediction | `Prokka`, `BUSCO` | Functional Interpretation. |

[Image of genome assembly and annotation pipeline workflow]

---

### 🖥️ Innovator HPC Initial Setup

Follow these steps to establish your workspace and build the necessary computational environments.

```bash
# 1. Sign in to the Innovator Cluster
ssh [your_username]@innovator.sdstate.edu

# 2. Navigate to your research space and create the project skeleton
# This follows a logical hierarchy to separate raw inputs from derived outputs.
mkdir -p genome_project/{data,scripts,envs,output,tutorials,logs}
cd genome_project

# 3. Load the Anaconda module available on Innovator
module load anaconda

# 4. Initialize conda for your shell (only needs to be done once)
conda init bash
source ~/.bashrc

# 5. Create the specialized Assembly environment
# This includes Raven for OLC assembly and NanoPolish for signal-level refinement
conda create -n assembly_env -c bioconda -c conda-forge \
    flye \
    raven-assembler \
    seqkit \
    nanopolish \
    racon \
    minimap2 \
    samtools -y

# 6. Create the Evaluation environment
conda create -n eval_env -c bioconda -c conda-forge \
    busco \
    quast \
    seqkit -y

# 7. Create the Circularization environment
conda create -n circulator_env -c bioconda -c conda-forge \
    circlator -y

# 8. Create the Annotation environment (including Enveomics/Evo2)
conda create -n annotation_env -c bioconda -c conda-forge \
    prokka \
    enveomics-python -y
```

On the Innovator HPC, managing resources is an exercise in **capacity planning**. You must balance the technical requirements of your tools with the available hardware to avoid "over-provisioning" (wasting resources) or "under-provisioning" (causing the tool to crash).

### 1. Surveying the Cluster with `sinfo`

The `sinfo` command is your "logical map" of the cluster. It tells you which partitions are available and how many nodes are currently idle.

* **To see all partitions:**
```bash
sinfo

```


* **To see detailed node status (Memory/CPUs):**
```bash
sinfo -o "%P %n %c %m %G %a"

```


*(Note: `%c` = CPUs, `%m` = Memory, `%G` = Generic Resources/GPUs)*

---

### 2. Launching an Interactive Session

For testing or pre-processing with `SeqKit`, you shouldn't run code on the login node. Instead, start an **Interactive Session**. This "borrows" a compute node's resources while keeping you in a live terminal.

```bash
# Requesting an interactive session on the 'comp' partition
srun --partition=comp --nodes=1 --cpus-per-task=4 --mem=16G --pty bash

```

* **`--pty bash`**: This is critical—it opens a "Pseudo-Terminal" so you can interact with the node.

---

### 3. Logic for Selecting CPUs, Memory, and GPUs

To select the appropriate amount of resources, you must look at the **algorithmic complexity** of the tool you are using.

#### **A. CPU Selection (Parallelization)**

* **Tool Type:** Does the tool support multi-threading (e.g., `flye --threads` or `prokka --cpus`)?
* **Strategy:** Most assemblers (`Flye`, `Raven`) scale well up to **16 or 32 CPUs**. Beyond that, you often hit "diminishing returns" where the overhead of managing threads outweighs the speed gain.
* **For*Calculation:** For bacterial genomes, **32GB to 64GB** is usually sufficient. For larger genomes, a rough estimate is ** the size of your input dataset**.

#### **C. GPU Selection**
mula:** If you have 1 million reads, 8–16 CPUs is usually a "sweet spot" for assembly.

#### **B. Memory (RAM) Selection**

* **The "Rule of Thumb":** Assembly is memory-intensive because the **Overlap Graph** must be stored in RAM.
* *
* **Tools:** Most traditional assemblers are **CPU-bound**. You only need a GPU for:
1. **Basecalling:** (e.g., `Guppy` or `Dorado`).
2. **Polishing:** Certain versions of `Medaka` use GPU acceleration (via TensorFlow) to speed up neural network inference.


* **Command:** Use `--gres=gpu:1` in your `sbatch` or `srun` command if the tool specifically supports CUDA.

---

### 📊 Resource Guide for your Workflow

| Tool | Recommended CPUs | RAM | GPU Needed? |
| --- | --- | --- | --- |
| **SeqKit** | 4–8 | 8GB | No |
| **Flye / Raven** | 16–32 | 64GB+ | No |
| **NanoPolish** | 8–16 | 32GB | No |
| **Medaka** | 4–8 | 16GB | **Yes (if available)** |
| **Prokka** | 8 | 16GB | No |

---

### 🔬 Mentee Challenge: Efficiency

Ask your mentee to run a small job and then use `seff [JOB_ID]` after it finishes.

* **The Goal:** If `seff` shows "CPU Efficiency: 10%," they requested too many CPUs.
* **The Logic:** Why is it ethically and technically better to request *exactly* what you need rather than "the maximum possible"? (Hint: Think about the queue wait times for other researchers).

**Would you like me to write a "Resource Monitor" script that the mentee can run during an interactive session to see their real-time RAM usage?**
