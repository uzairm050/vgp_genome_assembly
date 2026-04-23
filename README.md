# 🧬 VGP Genome Assembly — *Saccharomyces cerevisiae* S288C

> **A complete, chromosome-level genome assembly of *S. cerevisiae* S288C using the Vertebrate Genomes Project (VGP) pipeline on Galaxy (usegalaxy.eu).**

[![Galaxy](https://img.shields.io/badge/Platform-Galaxy%20usegalaxy.eu-1565C0?style=flat-square)](https://usegalaxy.eu)
[![Organism](https://img.shields.io/badge/Organism-S.%20cerevisiae%20S288C-2E7D32?style=flat-square)](https://www.ncbi.nlm.nih.gov/genome/15)
[![Genome Size](https://img.shields.io/badge/Genome%20Size-~11.7%20Mb-0F6E56?style=flat-square)](#results)
[![BUSCO Hap1](https://img.shields.io/badge/BUSCO%20Hap1-95.9%25-brightgreen?style=flat-square)](#busco--gene-completeness)
[![Merqury](https://img.shields.io/badge/Merqury-99.9999%25-brightgreen?style=flat-square)](#merqury--k-mer-quality)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Pipeline Summary](#pipeline-summary)
- [Input Data](#input-data)
- [Tools Used](#tools-used)
- [Step-by-Step Workflow](#step-by-step-workflow)
  - [1. Data Upload & Preprocessing](#1-data-upload--preprocessing)
  - [2. Genome Profile Analysis](#2-genome-profile-analysis)
  - [3. De Novo Assembly with Hifiasm](#3-de-novo-assembly-with-hifiasm)
  - [4. Assembly Quality Control](#4-assembly-quality-control)
  - [5. Scaffolding with Bionano Optical Maps](#5-scaffolding-with-bionano-optical-maps)
  - [6. Scaffolding with Hi-C (YaHS)](#6-scaffolding-with-hi-c-yahs)
  - [7. Final Evaluation](#7-final-evaluation)
- [Results](#results)
- [GenomeScope2 — Genome Profile](#genomescope2--genome-profile)
- [BUSCO — Gene Completeness](#busco--gene-completeness)
- [Merqury — K-mer Quality](#merqury--k-mer-quality)
- [Hi-C Contact Maps](#hi-c-contact-maps)
- [Repository Structure](#repository-structure)
- [Glossary](#glossary)
- [References](#references)

---

## Overview

This repository documents the complete **de novo genome assembly** of *Saccharomyces cerevisiae* strain S288C using the **Vertebrate Genomes Project (VGP)** pipeline on [Galaxy (usegalaxy.eu)](https://usegalaxy.eu), following the [Galaxy Training Network VGP tutorial](https://training.galaxyproject.org/training-material/topics/assembly/tutorials/vgp_genome_assembly/tutorial.html).

The VGP pipeline integrates three complementary sequencing technologies to produce a high-quality, phased, chromosome-level assembly:

| Technology | Platform | Role |
|---|---|---|
| **PacBio HiFi reads** | SMRT sequencing (>Q20, 10–25 kb) | Primary long-read assembly input |
| **Illumina Hi-C reads** | Chromatin conformation capture | Haplotype phasing + chromosome-scale scaffolding |
| **Bionano optical maps** | Nanochannel DNA imaging | Long-range physical-map scaffolding |

*S. cerevisiae* S288C (~12 Mb haploid genome, 16 chromosomes) is an ideal benchmarking organism due to its well-characterized reference genome and compact size.

---

## Pipeline Summary

```
RAW INPUTS
├── HiFi reads  (3 × FASTA, 50× depth)
├── Hi-C reads  (Forward + Reverse FASTQ)
└── Bionano map (bionano.cmap)
         │
         ▼
PHASE 1 — PREPARE
  Step 1  │ Cutadapt       → Remove adapter-containing reads
  Step 2  │ Meryl ×3       → Count k-mers → merge → histogram
  Step 3  │ GenomeScope2   → Estimate genome size & heterozygosity
         │
         ▼
PHASE 2 — ASSEMBLE
  Step 4  │ Hifiasm        → De novo diploid assembly (Hi-C phased mode)
  Step 5  │ Gfastats       → GFA → FASTA + assembly statistics
         │
         ▼
PHASE 3 — QUALITY CONTROL
  Step 6  │ BUSCO          → Gene completeness (2,137 Saccharomycetes genes)
  Step 7  │ Merqury        → K-mer completeness + phasing quality
         │
         ▼
PHASE 4 — SCAFFOLD
  Step 8    │ Bionano Scaffold         → Optical-map scaffolding
  Step 9–10 │ BWA-MEM ×2 + Filter     → Map & filter Hi-C reads
  Step 11   │ PretextMap + Snapshot   → Pre-scaffolding contact map
  Step 12   │ YaHS                    → Chromosome-level scaffolding
  Step 13–14│ Remap + Final Pretext   → Final Hi-C contact map
         │
         ▼
  Chromosome-level genome assembly (~11.7 Mb, 16 chromosomes) ✅
```

---

## Input Data

All input data is publicly available from Zenodo:

| File | Type | Zenodo Record | Purpose |
|---|---|---|---|
| `HiFi_synthetic_50x_01/02/03.fasta` | FASTA | [6098306](https://zenodo.org/record/6098306) | PacBio HiFi reads (50×, synthetic) |
| `SRR7126301_1.fastq.gz` | FASTQ | [5550653](https://zenodo.org/record/5550653) | Hi-C forward reads |
| `SRR7126301_2.fastq.gz` | FASTQ | [5550653](https://zenodo.org/record/5550653) | Hi-C reverse reads |
| `bionano.cmap` | CMAP | [5887339](https://zenodo.org/record/5887339) | Bionano optical map |

> **Note:** The HiFi reads are **synthetic** — simulated from the *S. cerevisiae* S288C reference genome using HIsim at 50× coverage with a 2% mutation rate to mimic a heterozygous diploid.

---

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| [Cutadapt](https://cutadapt.readthedocs.io) | 5.2+galaxy2 | Adapter trimming / chimeric read removal |
| [Meryl](https://github.com/marbl/meryl) | 1.3+galaxy6 | K-mer counting and histogram generation |
| [GenomeScope2](https://github.com/tbenavi1/genomescope2.0) | 2.0+galaxy2 | Genome size and heterozygosity estimation |
| [Hifiasm](https://github.com/chhylp123/hifiasm) | 0.19.8+galaxy0 | De novo haplotype-phased assembly |
| [Gfastats](https://github.com/vgl-hub/gfastats) | 1.3.6+galaxy0 | Assembly statistics + GFA→FASTA conversion |
| [BUSCO](https://busco.ezlab.org) | 5.5.0+galaxy0 | Gene completeness assessment |
| [Merqury](https://github.com/marbl/merqury) | 1.3+galaxy3 | K-mer QV and phasing quality |
| [Bionano Hybrid Scaffold](https://bionano.com) | 3.7.0+galaxy3 | Optical map scaffolding |
| [BWA-MEM](https://github.com/lh3/bwa) | 2.2.1+galaxy1 | Hi-C read mapping |
| [Filter and merge](https://github.com/ArimaGenomics/mapping_pipeline) | 1.0+galaxy1 | Chimeric Hi-C read filtering |
| [PretextMap](https://github.com/wtsi-hpag/PretextMap) | 0.1.9+galaxy0 | Hi-C contact map generation |
| [Pretext Snapshot](https://github.com/wtsi-hpag/PretextSnapshot) | 0.0.3+galaxy1 | Contact map image export |
| [YaHS](https://github.com/c-zhou/yahs) | 1.2a.2+galaxy1 | Hi-C chromosome-scale scaffolding |

---

## Step-by-Step Workflow

### 1. Data Upload & Preprocessing

**Objective:** Upload all datasets to Galaxy and remove adapter-contaminated HiFi reads.

The three HiFi FASTA files were uploaded to Galaxy and grouped into a **dataset collection** named `HiFi data`, enabling parallel processing across all files.

Cutadapt was run to **discard** any reads containing adapter sequences (not trim — fully remove):

| Parameter | Value |
|---|---|
| Mode | Single-end |
| Adapter 1 | `ATCTCTCTCAACAACAACAACGGAGGAGGAGGAAAAGAGAGAGAT` |
| Adapter 2 | `ATCTCTCTCTTTTCCTCCTCCTCCGTTGTTGTTGTTGAGAGAGAT` |
| Max error rate | 0.1 |
| Min overlap | 35 |
| Reverse complement search | Yes |
| Discard reads with adapters | Yes |

> **Why discard instead of trim?** PacBio SMRT sequencing can embed adapters *anywhere* within a read — not just at the ends. Any read containing an adapter is therefore likely chimeric and must be removed entirely rather than end-trimmed.

**Output:** `HiFi_collection (trimmed)`

---

### 2. Genome Profile Analysis

**Objective:** Estimate genome size, heterozygosity, and repeat content from k-mer frequencies *before* assembly begins.

#### 2a. Meryl — K-mer Counting

Meryl was run in three sequential steps:
1. **Count** k-mers (k=31) in each trimmed HiFi file individually (parallelized via collection)
2. **Merge** all k-mer databases with `Union-sum` into a single combined database
3. **Generate a histogram** from the merged database

> k=31 is long enough to avoid most repetitive k-mers while remaining robust to sequencing errors — the standard choice for mammalian/fungal genomes.

#### 2b. GenomeScope2 — Genome Profiling

GenomeScope2 fit a diploid model (ploidy=2, k=31) to the k-mer histogram.

| Parameter | Estimated Value |
|---|---|
| Haploid genome size | ~11.7 Mb |
| Heterozygosity | ~0.576% |
| Diploid coverage | ~50× |
| Haploid coverage | ~25× |
| Model fit | >93% |

The bimodal k-mer distribution (peaks at ~25× and ~50×) is the expected signature of a diploid genome. These estimates were used to parameterize all downstream tools.

---

### 3. De Novo Assembly with Hifiasm

**Objective:** Produce two fully phased haplotype assemblies from HiFi + Hi-C reads.

**Mode used:** Hi-C phased mode (`--hic`)

This mode uses Hi-C data to phase contigs into two distinct haplotypes (Hap1 and Hap2) rather than a collapsed pseudohaplotype. The Hi-C reads physically link heterozygous variants, allowing Hifiasm to assign each contig to its correct parental chromosome copy.

| Parameter | Value |
|---|---|
| Assembly mode | Standard |
| Input reads | `HiFi_collection (trimmed)` |
| Hi-C R1 | `Hi-C_dataset_F` |
| Hi-C R2 | `Hi-C_dataset_R` |

**Outputs:**
- `Hap1 contigs graph` (GFA format)
- `Hap2 contigs graph` (GFA format)

GFA files were converted to FASTA using Gfastats (path generation enabled):
- `Hap1 contigs FASTA`
- `Hap2 contigs FASTA`

---

### 4. Assembly Quality Control

**Objective:** Evaluate assembly quality using three complementary, independent metrics.

#### 4a. Gfastats — Contig Statistics

Gfastats was run on both haplotype GFA files with expected genome size = **11,747,160 bp**.

| Statistic | Hap1 | Hap2 |
|---|---|---|
| # contigs | 16 | 17 |
| Total length | ~11.3 Mb | ~12.2 Mb |
| N50 | ~922 kb | ~923 kb |

#### 4b. BUSCO — Gene Completeness

BUSCO searched for 2,137 conserved genes expected in every *Saccharomycetes* genome (Metaeuk mode).

- **Hap1:** `C:95.9% [S:94.1%, D:1.8%], F:2.7%, M:1.4%` ✅
- **Hap2:** `C:89.0% [S:87.6%, D:1.4%], F:2.5%, M:8.5%`

High completeness (>95%) and low duplication (<2%) for Hap1 confirm a clean, high-quality assembly. The slightly lower Hap2 completeness is expected for the alternate haplotype.

#### 4c. Merqury — K-mer Quality

Merqury compared k-mers in the HiFi reads against both assemblies — entirely reference-free.

- **Spectra-cn plot:** Diploid coverage peak at ~50× consistent with GenomeScope2
- **Spectra-asm plot:** Haploid k-mers (~25×) correctly split between Hap1 and Hap2; homozygous k-mers (~50×) shared by both — the expected pattern for a well-phased diploid
- **Combined completeness: 99.9999%**

---

### 5. Scaffolding with Bionano Optical Maps

**Objective:** Use physical restriction-map data to merge and orient contigs into longer scaffolds, resolving structural regions that sequence data alone cannot span.

The Bionano Hybrid Scaffold tool integrates in-silico restriction maps derived from the contig sequences with experimentally derived Bionano genome maps through five internal steps:

1. Generate in-silico sequence maps from contigs
2. Align against Bionano genome maps and detect conflicts
3. Merge non-conflicting maps into hybrid scaffolds
4. Align sequence maps back to hybrid scaffolds
5. Output AGP and FASTA files

| Parameter | Value |
|---|---|
| Configuration mode | VGP mode |
| Genome maps conflict filter | Cut contig at conflict |
| Sequences conflict filter | Cut contig at conflict |

After scaffolding, the `NGScontigs scaffold` output and unscaffolded contigs were concatenated to produce: `Hap1 assembly bionano`

---

### 6. Scaffolding with Hi-C (YaHS)

**Objective:** Use chromatin conformation data to order and orient Bionano scaffolds to chromosome scale.

#### 6a. Hi-C Read Mapping

Hi-C reads were mapped **individually** (not as pairs) against `Hap1 assembly bionano` using BWA-MEM, sorted by read name (QNAME), then filtered for chimeric reads using Filter and merge.

> Hi-C reads *must* be mapped individually because the genomic insert size between paired Hi-C reads can range from 1 bp to hundreds of Mb — violating the fixed-insert-size assumptions of standard paired-end aligners.

**Output:** `BAM Hi-C reads`

#### 6b. Pre-Scaffolding Contact Map

PretextMap and Pretext Snapshot were used to generate a pre-YaHS Hi-C contact map for baseline comparison.

#### 6c. YaHS Scaffolding

| Parameter | Value |
|---|---|
| Restriction enzyme | `CTTAAG` (HindIII) |
| Input contigs | `Hap1 assembly bionano` |
| Hi-C alignment | `BAM Hi-C reads` |

**Output:** `YaHS Scaffolds FASTA`

YaHS performs multi-round hierarchical scaffolding at decreasing resolution and includes built-in assembly error correction based on Hi-C coverage anomalies — it does not require a pre-specified chromosome count.

---

### 7. Final Evaluation

Hi-C reads were remapped to the YaHS scaffolds and a final contact map was generated for comparison against the pre-scaffolding map.

**Key observations:**
- **16–17 distinct triangular blocks** visible on the diagonal — each block = one chromosome ✅
- All inter-chromosomal contact signal absent (off-diagonal noise-free)
- Inversions corrected by YaHS visible when comparing before/after maps
- Final assembly closely matches *S. cerevisiae* S288C reference genome topology

---

## Results

| Metric | Value |
|---|---|
| Estimated genome size | ~11.7 Mb |
| Genome heterozygosity | 0.58% |
| Hap1 contig count | 16 |
| Hap2 contig count | 17 |
| Hap1 contig N50 | ~922 kb |
| Hap2 contig N50 | ~923 kb |
| BUSCO completeness (Hap1) | **95.9%** |
| BUSCO completeness (Hap2) | 89.0% |
| Merqury combined completeness | **99.9999%** |
| Scaffolding level | Chromosome-level |
| Chromosomes in final map | 16 |

---

## GenomeScope2 — Genome Profile

GenomeScope2 fits a mathematical model to the k-mer histogram to estimate genome properties **before assembly begins**.

### Linear Profile

![GenomeScope2 linear plot](figures/genomescope/genomescope_linear_plot.png)

> **Reading this plot:** First peak at ~25× = heterozygous k-mers (one chromosome copy). Second peak at ~50× = homozygous k-mers (both copies). Orange = sequencing errors. Black model line fits blue observed data at **96.5%** — results are trustworthy.

### Transformed Linear Profile

![GenomeScope2 transformed plot](figures/genomescope/genomescope_transformed_linear_plot.png)

> Multiplying frequency × coverage amplifies the homozygous peak, making the diploid structure clearer. Reports: `len=11,743,432 bp`, `ab=0.58%`, `aa=99.4%`, `kcov=25`, `err=0.000943%`.

### Summary Statistics

```
Property                       Min               Max
Homozygous (aa)                99.4165%          99.4241%
Heterozygous (ab)              0.575891%         0.583546%
Genome Haploid Length          11,739,513 bp     11,747,352 bp
Genome Unique Length           11,016,399 bp     11,023,756 bp
Model Fit                      92.5159%          96.5191%
Read Error Rate                0.000943190%      0.000943190%
```

---

## BUSCO — Gene Completeness

BUSCO searches for 2,137 conserved genes expected in every *Saccharomycetes* genome. High completeness (>95%) and low duplication (<5%) confirm a high-quality assembly with no false duplications.

### Hap1 — 95.9% Complete

![BUSCO Hap1](figures/busco/busco_hap1.png)

> `C:95.9% [S:94.1%, D:1.8%], F:2.7%, M:1.4%, n:2137` — Almost entirely complete single-copy genes (light blue). Very small duplicated (dark blue) and missing (red) fractions confirm an excellent assembly.

### Hap2 — 89.0% Complete

![BUSCO Hap2](figures/busco/busco_hap2.png)

> `C:89.0% [S:87.6%, D:1.4%], F:2.5%, M:8.5%, n:2137` — Slightly lower completeness is expected for the alternate haplotype. Low duplication (1.4%) confirms no false duplications.

### Comparison Table

| Category | Hap1 | Hap1 % | Hap2 | Hap2 % |
|---|---|---|---|---|
| Complete single-copy (S) | 2010 | 94.1% | 1871 | 87.6% |
| Complete duplicated (D) | 40 | 1.8% | 31 | 1.4% |
| **Complete total (C)** | **2050** | **95.9%** | **1902** | **89.0%** |
| Fragmented (F) | 57 | 2.7% | 54 | 2.5% |
| Missing (M) | 30 | 1.4% | 181 | 8.5% |

---

## Merqury — K-mer Quality

Merqury compares k-mers in reads vs assembly entirely reference-free. Combined completeness of **99.9999%** means virtually every k-mer in the reads is represented in one of the two assemblies.

### Completeness Statistics

| Assembly | K-mers found | Total k-mers | Completeness |
|---|---|---|---|
| Hap1 | 10,792,811 | 13,010,260 | 82.96% |
| Hap2 | 11,611,483 | 13,010,260 | 89.25% |
| **Both** | **13,010,244** | **13,010,260** | **99.9999%** |

### ASM Spectrum — Phasing Quality

![Merqury ASM spectrum](figures/merqury/merqury_asm_spectrum_filled.png)

> **Green** = k-mers shared between Hap1+Hap2 (homozygous regions). **Red** = Hap1-only. **Blue** = Hap2-only. Large green peak at ~50× + balanced red/blue peaks at ~25× = **excellent phasing**. ✅

### Hap1 CN Spectrum

![Merqury Hap1 CN spectrum](figures/merqury/merqury_hap1_cn_spectrum_filled.png)

> **Red** = 1-copy k-mers (heterozygous). **Blue** = 2-copy k-mers (homozygous). **Grey** = k-mers present in reads but absent from Hap1 (correctly assigned to Hap2).

### Hap2 CN Spectrum

![Merqury Hap2 CN spectrum](figures/merqury/merqury_hap2_cn_spectrum_filled.png)

---

## Hi-C Contact Maps

Hi-C contact maps reveal which genomic regions physically interact in the cell nucleus. Regions on the **same chromosome** interact far more than regions on **different chromosomes**. Each triangular block along the diagonal represents **one chromosome**.

### Before YaHS Scaffolding

Generated from Hi-C reads mapped to Bionano-scaffolded contigs. The thin diagonal confirms contigs are internally correct but not yet ordered into chromosomes.

![Before YaHS — full view](figures/hic_maps/before_yahs/hic_map_before_01_full.png)

### After YaHS Scaffolding

After YaHS, the contact map shows **16 clear triangular blocks** — each corresponding to one *S. cerevisiae* chromosome. This confirms successful chromosome-level assembly. ✅

![After YaHS — full view](figures/hic_maps/after_yahs/hic_map_after_01_full.png)

### Before vs After Comparison

| Before YaHS (Step 11) | After YaHS (Step 14) |
|---|---|
| Thin diagonal — contigs not yet in chromosome order | 16 clear blocks — each = one chromosome ✅ |

---

## Repository Structure

```
vgp_genome_assembly/
│
├── README.md                           ← This file
│
├── docs/
│   ├── 01_pipeline_overview.md         ← Biology and rationale for each phase
│   ├── 02_input_output_table.md        ← Every step: input → tool → output
│   ├── 03_tool_parameters.md           ← Exact settings for every tool
│   └── 04_results_interpretation.md    ← How to read BUSCO, Merqury, contact maps
│
├── results/
│   ├── genomescope_summary.txt         ← GenomeScope2 output
│   ├── assembly_stats.txt              ← Gfastats contig statistics
│   ├── busco_hap1_summary.txt          ← BUSCO Hap1 (95.9%)
│   ├── busco_hap2_summary.txt          ← BUSCO Hap2 (89.0%)
│   └── merqury_completeness.txt        ← Merqury (99.9999%)
│
├── figures/
│   ├── genomescope/                    ← GenomeScope2 profile plots
│   ├── busco/                          ← BUSCO bar charts
│   ├── merqury/                        ← Merqury spectrum plots
│   ├── hic_maps/
│   │   ├── before_yahs/                ← Pre-scaffolding contact maps
│   │   └── after_yahs/                 ← Post-scaffolding contact maps
│   └── galaxy_screenshots/             ← Galaxy workflow screenshots
│
└── scripts/
    └── upload_urls.txt                 ← All Zenodo URLs for input data
```

---

## Glossary

| Term | Definition |
|---|---|
| **Contig** | A contiguous, gap-free assembled DNA sequence |
| **Scaffold** | One or more contigs joined by gap sequences (Ns) |
| **Haplotype** | One version of a chromosome in a diploid organism |
| **Phasing** | Assigning contigs to their correct parental haplotype |
| **HiFi reads** | PacBio reads 10–25 kbp long with ≥99% base accuracy |
| **k-mer** | A DNA substring of fixed length k |
| **N50** | Contig length at which ≥50% of the total assembly is in contigs of that size or longer |
| **BUSCO** | Benchmarking Universal Single-Copy Orthologs — a gene completeness metric |
| **QV** | Quality value — phred-scale measure of per-base assembly accuracy |
| **Hi-C** | Chromatin conformation capture sequencing — captures 3D genome organization |
| **Optical map** | Long-range physical genome map generated by restriction enzyme labeling |
| **GFA** | Graphical Fragment Assembly format — encodes assembly graphs |
| **T2T** | Telomere-to-Telomere — a completely gapless chromosome assembly |
| **Haplotig** | A contig representing one haplotype in a diploid assembly |

---

## References

1. **Rhie A. et al.** (2021). Towards complete and error-free genome assemblies of all vertebrate species. *Nature*, 592, 737–746. https://doi.org/10.1038/s41586-021-03451-0
2. **Cheng H. et al.** (2021). Haplotype-resolved de novo assembly using phased assembly graphs with hifiasm. *Nature Methods*, 18, 170–175. https://doi.org/10.1038/s41592-020-01056-5
3. **Rhie A. et al.** (2020). Merqury: reference-free quality, completeness, and phasing assessment for genome assemblies. *Genome Biology*, 21, 245. https://doi.org/10.1186/s13059-020-02134-9
4. **Zhou C., McCarthy S.A., Durbin R.** (2022). YaHS: yet another Hi-C scaffolding tool. *Bioinformatics*, 39, btac808. https://doi.org/10.1093/bioinformatics/btac808
5. **Simão F.A. et al.** (2015). BUSCO: assessing genome assembly and annotation completeness with single-copy orthologs. *Bioinformatics*, 31, 3210–3212. https://doi.org/10.1093/bioinformatics/btv351
6. **Formenti G. et al.** (2022). Gfastats: conversion, evaluation and manipulation of genome sequences. *Bioinformatics*, 38, 4214–4216. https://doi.org/10.1093/bioinformatics/btac460
7. **Ranallo-Benavidez T.R. et al.** (2020). GenomeScope 2.0 and Smudgeplots for reference-free profiling of polyploid genomes. *Nature Communications*, 11, 1432. https://doi.org/10.1038/s41467-020-14998-3
8. **Wenger A.M. et al.** (2019). Accurate circular consensus long-read sequencing improves variant detection and assembly of a human genome. *Nature Biotechnology*, 37, 1155–1162. https://doi.org/10.1038/s41587-019-0217-9
9. **Galaxy Training Network.** VGP genome assembly tutorial. https://training.galaxyproject.org/training-material/topics/assembly/tutorials/vgp_genome_assembly/tutorial.html

---

*Completed as part of the MS Bioinformatics program, NUST Islamabad. Pipeline executed on [Galaxy (usegalaxy.eu)](https://usegalaxy.eu).*
