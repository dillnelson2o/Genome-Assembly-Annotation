
```markdown
# 🛠 Pre-Assembly Tutorial: Long-Read Curation

Before we trigger the mechanistic reconstruction of the genome, we must perform **Quality Control (QC)** and **Read Manipulation**. Our goal is to increase the **Mean Read Quality** and **N50 length** of our input data.

---

## 1. Resource Planning on Innovator HPC
Pre-assembly tasks like `SeqKit` are fast but can be memory-intensive if processing large files. We will use an **Interactive Session** for this work.

**How to request resources:**
```

```bash
# Request 8 CPUs and 32GB of RAM for 2 hours
srun --partition=comp --nodes=1 --cpus-per-task=8 --mem=32G --pty bash

```

*Note: Use `sinfo` to ensure the `comp` partition has available nodes.*

---

## 2. Environment Activation

We will use the `assembly_env` created in the main setup, as it contains `SeqKit` and `NanoPlot`.

```bash
module load anaconda
source activate assembly_env

```

---

## 3. Initial Data Interrogation

We start with **inductive observation**. We need to know what our raw data looks like before we decide on filtering thresholds.

### Statistical Summary with SeqKit

```bash
seqkit stats data/*.fastq.gz

```

### Visual QC with NanoPlot

NanoPlot provides a "topological view" of your read lengths versus their quality scores.

```bash
NanoPlot --fastq data/*.fastq.gz -o output/QC_reports/raw_data --threads 8

```

* **Logic:** Look for the "N50" of your reads. If your N50 is < 5kb, your assembly may struggle to resolve repeats.

---

## 4. Deductive Filtering & Processing

Now we apply "rules" to our data to remove noise.

### A. Length & Quality Filtering

We want to remove "ultra-short" reads (which add noise) and "low-quality" reads (which introduce errors).

```bash
# Filter: Keep reads > 1000bp AND mean quality > Q10
seqkit seq -m 1000 -Q 10 data/raw_reads.fastq.gz > data/filtered_reads.fastq

```

### B. Subsampling (Optional)

If you have  coverage, the assembly graph becomes overly complex. We can use **probabilistic subsampling** to reduce the dataset to a more manageable size (e.g., ).

```bash
# Subsample to 10% of total reads
seqkit sample -p 0.1 data/filtered_reads.fastq > data/subsampled_reads.fastq

```

---

## ⚙️ The Mechanics: Why Pre-Process?

### The "Chimeric Read" Problem

During Nanopore sequencing, two distinct DNA molecules can sometimes pass through the pore so closely that the software calls them as a single "chimeric" read.

* **The Danger:** If a read contains pieces from two different parts of the genome, the assembler will create a "false bridge," leading to a collapsed or incorrect assembly.
* **The Solution:** Tools like `Raven` or `Flye` have internal "chimeric detection," but pre-filtering by length/quality reduces the mathematical burden on the assembler.

---

## 🔬 Mentee Challenge: Threshold Logic

1. **Trade-offs:** If you set your filter to `-m 10000` (only keep reads > 10kb), you might have a very "clean" assembly, but you might lose **coverage**. If coverage drops below , you may not have enough data to close the genome.
2. **Observation:** Run `seqkit stats` on your file *before* and *after* filtering. What percentage of your total "Gbp" (Giga-base pairs) did you lose? Is that loss acceptable for the gain in quality?

---

### Next Steps

Once your data is filtered, proceed to the **Assembly** phase using `Flye` or `Raven`. Would you like a SLURM template for running `Raven` specifically?

```

### Logic Check for your HPC setup:
* **NanoPlot:** I added this to the instructions because it’s the standard for ONT visualization. You may need to add it to your `assembly_env` using `conda install -c bioconda nanoplot`.
* **Interactive vs Batch:** Remind the mentee that `SeqKit` commands are fast enough for `srun`, but `NanoPlot` and `Flye` should usually be `sbatch` because they take longer.

