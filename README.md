# 🧬 VGP Genome Assembly — *Saccharomyces cerevisiae* S288C

> **A complete, chromosome-level genome assembly using the Vertebrate Genomes Project (VGP) pipeline on Galaxy.**

[![Galaxy](https://img.shields.io/badge/Platform-Galaxy%20usegalaxy.eu-1565C0?style=flat-square)](https://usegalaxy.eu)
[![Organism](https://img.shields.io/badge/Organism-S.%20cerevisiae%20S288C-2E7D32?style=flat-square)](https://www.ncbi.nlm.nih.gov/genome/15)
[![Genome Size](https://img.shields.io/badge/Genome%20Size-~11.7%20Mb-0F6E56?style=flat-square)](#results)
[![BUSCO Hap1](https://img.shields.io/badge/BUSCO%20Hap1-95.9%25-brightgreen?style=flat-square)](#busco--gene-completeness)
[![Merqury](https://img.shields.io/badge/Merqury-99.9999%25-brightgreen?style=flat-square)](#merqury--k-mer-quality)

## 📋 Table of Contents

- [Overview](#overview)
- [Pipeline Summary](#pipeline-summary)
- [Input Data](#input-data)
- [Tools Used](#tools-used)
- [Results](#results)
- [GenomeScope2 — Genome Profile](#genomescope2--genome-profile)
- [BUSCO — Gene Completeness](#busco--gene-completeness)
- [Merqury — K-mer Quality](#merqury--k-mer-quality)
- [Hi-C Contact Maps](#hi-c-contact-maps)
- [Galaxy Workflow Screenshots](#galaxy-workflow-screenshots)
- [Repository Structure](#repository-structure)
- [References](#references)

---

## Overview

This repository documents a complete **de novo genome assembly** of *Saccharomyces cerevisiae* strain S288C using the **Vertebrate Genomes Project (VGP)** pipeline on [Galaxy (usegalaxy.eu)](https://usegalaxy.eu).

The VGP pipeline combines three complementary data types:

| Data Type | Technology | Role |
|---|---|---|
| **PacBio HiFi reads** | SMRT (>Q20, 10–25 kb) | Main assembly input |
| **Illumina Hi-C reads** | Chromatin conformation capture | Haplotype phasing + scaffolding |
| **Bionano optical maps** | Nanochannel DNA imaging | Physical-map scaffolding |

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
  Step 8  │ Bionano Scaffold  → Optical-map scaffolding
  Step 9-10 │ BWA-MEM ×2 + Filter & Merge  → Map Hi-C reads
  Step 11 │ PretextMap + Snapshot  → Initial Hi-C contact map
  Step 12 │ YaHS           → Chromosome-level scaffolding
  Step 13-14 │ Remap + Final Pretext  → Final contact map
         │
         ▼
  Chromosome-level genome assembly (~11.7 Mb, 16 chromosomes)
```

---

## Input Data

All input data sourced from Zenodo:

| File | Type | Zenodo | Purpose |
|---|---|---|---|
| `HiFi_synthetic_50x_01/02/03.fasta` | FASTA | [6098306](https://zenodo.org/record/6098306) | PacBio HiFi reads (50×) |
| `SRR7126301_1.fastq.gz` | FASTQ | [5550653](https://zenodo.org/record/5550653) | Hi-C forward reads |
| `SRR7126301_2.fastq.gz` | FASTQ | [5550653](https://zenodo.org/record/5550653) | Hi-C reverse reads |
| `bionano.cmap` | CMAP | [5887339](https://zenodo.org/record/5887339) | Bionano optical map |

> The HiFi reads are **synthetic** — simulated from the *S. cerevisiae* S288C reference genome using HIsim at 50× coverage with a 2% mutation rate.

---

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| [Cutadapt](https://cutadapt.readthedocs.io) | 5.2+galaxy2 | Adapter trimming |
| [Meryl](https://github.com/marbl/meryl) | 1.3+galaxy6 | K-mer counting |
| [GenomeScope2](https://github.com/tbenavi1/genomescope2.0) | 2.0+galaxy2 | Genome profiling |
| [Hifiasm](https://github.com/chhylp123/hifiasm) | 0.19.8+galaxy0 | De novo assembly |
| [Gfastats](https://github.com/vgl-hub/gfastats) | 1.3.6+galaxy0 | Assembly stats |
| [BUSCO](https://busco.ezlab.org) | 5.5.0+galaxy0 | Gene completeness |
| [Merqury](https://github.com/marbl/merqury) | 1.3+galaxy3 | K-mer quality |
| [Bionano Hybrid Scaffold](https://bionano.com) | 3.7.0+galaxy3 | Optical map scaffolding |
| [BWA-MEM](https://github.com/lh3/bwa) | 2.2.1+galaxy1 | Hi-C read mapping |
| [Filter and merge](https://github.com/ArimaGenomics/mapping_pipeline) | 1.0+galaxy1 | Chimeric read filtering |
| [PretextMap](https://github.com/wtsi-hpag/PretextMap) | 0.1.9+galaxy0 | Contact map generation |
| [Pretext Snapshot](https://github.com/wtsi-hpag/PretextSnapshot) | 0.0.3+galaxy1 | Contact map image |
| [YaHS](https://github.com/c-zhou/yahs) | 1.2a.2+galaxy1 | Hi-C scaffolding |

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
| Chromosomes visible in final map | 16 |

---

## GenomeScope2 — Genome Profile

GenomeScope2 fits a mathematical model to the k-mer histogram to estimate genome properties **before assembly begins**.

### Linear Profile

![GenomeScope2 linear plot](figures/genomescope/genomescope_linear_plot.png)

> **Reading this plot:** First peak at ~25× = heterozygous k-mers (one chromosome copy). Second peak at ~50× = homozygous k-mers (both copies). Orange = sequencing errors. Black model line fits blue observed data at **96.5%** — results are trustworthy.

### Transformed Linear Profile

![GenomeScope2 transformed plot](figures/genomescope/genomescope_transformed_linear_plot.png)

> Multiplying frequency × coverage amplifies the homozygous peak, making the diploid structure easier to see. Reports: `len=11,743,432 bp`, `ab=0.58%`, `aa=99.4%`, `kcov=25`, `err=0.000943%`.

### Log Scale Views

| Linear log | Transformed log |
|---|---|
| ![Log plot](figures/genomescope/genomescope_log_plot.png) | ![Transformed log](figures/genomescope/genomescope_transformed_log_plot.png) |

### Summary Statistics

```
property                      min               max
Homozygous (aa)               99.4165%          99.4241%
Heterozygous (ab)             0.575891%         0.583546%
Genome Haploid Length         11,739,513 bp     11,747,352 bp
Genome Unique Length          11,016,399 bp     11,023,756 bp
Model Fit                     92.5159%          96.5191%
Read Error Rate               0.000943190%      0.000943190%
```

---

## BUSCO — Gene Completeness

BUSCO searches for 2,137 conserved genes expected in every *Saccharomycetes* genome. High completeness (>95%) and low duplication (<5%) confirm a high-quality assembly with no false duplications.

### Hap1 — 95.9% Complete

![BUSCO Hap1](figures/busco/busco_hap1.png)

> `C:95.9% [S:94.1%, D:1.8%], F:2.7%, M:1.4%, n:2137` — The bar is almost entirely light blue (complete single-copy genes). Very small dark blue (duplicated) and red (missing) segments confirm an excellent assembly.

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

Merqury compares k-mers in reads vs assembly — completely reference-free. Combined completeness of **99.9999%** means virtually every k-mer in the reads is represented in one of the two assemblies.

### Completeness Statistics

| Assembly | K-mers found | Total k-mers | Completeness |
|---|---|---|---|
| Hap1 | 10,792,811 | 13,010,260 | 82.96% |
| Hap2 | 11,611,483 | 13,010,260 | 89.25% |
| **Both** | **13,010,244** | **13,010,260** | **99.9999%** |

### Hap1 CN Spectrum

![Merqury Hap1 CN spectrum](figures/merqury/merqury_hap1_cn_spectrum_filled.png)

> **Red** = 1-copy k-mers (heterozygous). **Blue** = 2-copy k-mers (homozygous). **Grey** = k-mers in reads but not in this assembly (present in Hap2 instead — expected).

### Hap2 CN Spectrum

![Merqury Hap2 CN spectrum](figures/merqury/merqury_hap2_cn_spectrum_filled.png)

### ASM Spectrum — Phasing Quality (most important plot)

![Merqury ASM spectrum](figures/merqury/merqury_asm_spectrum_filled.png)

> **Green** = k-mers shared between Hap1+Hap2 (homozygous regions). **Red** = Hap1-only. **Blue** = Hap2-only. Large green peak at ~50× + balanced red/blue peaks at ~25× = **excellent phasing**. ✅

### Combined CN Spectrum

![Merqury combined CN spectrum](figures/merqury/merqury_combined_cn_spectrum.png)

### Additional Spectrum Plots

<details>
<summary>View all Merqury spectrum plots (line versions + additional views)</summary>

| Hap1 CN (line) | Hap2 CN (line) |
|---|---|
| ![](figures/merqury/merqury_hap1_cn_spectrum_line.png) | ![](figures/merqury/merqury_hap2_cn_spectrum_line.png) |

| ASM spectrum (line) | Both CN (filled) |
|---|---|
| ![](figures/merqury/merqury_asm_spectrum_line.png) | ![](figures/merqury/merqury_both_cn_spectrum_filled.png) |

| Both CN (line) | Both CN (partial) |
|---|---|
| ![](figures/merqury/merqury_both_cn_spectrum_line.png) | ![](figures/merqury/merqury_both_cn_spectrum_partial.png) |

</details>

---

## Hi-C Contact Maps

Hi-C contact maps show genomic regions that physically interact in the cell nucleus. Regions on the **same chromosome** interact far more than regions on **different chromosomes**. Each triangular block along the diagonal = **one chromosome**.

### Before YaHS Scaffolding (Step 11)

Generated from Hi-C reads mapped to the Bionano-scaffolded contigs. The thin diagonal confirms contigs are internally correct, but they are not yet assembled into chromosomes.

![Before YaHS — full view](figures/hic_maps/before_yahs/hic_map_before_01_full.png)

| Detail view 1 | Detail view 2 |
|---|---|
| ![](figures/hic_maps/before_yahs/hic_map_before_07.png) | ![](figures/hic_maps/before_yahs/hic_map_before_08.png) |

| Detail view 3 | Detail view 4 |
|---|---|
| ![](figures/hic_maps/before_yahs/hic_map_before_09.png) | ![](figures/hic_maps/before_yahs/hic_map_before_10.png) |

<details>
<summary>View all 17 pre-scaffolding contact maps</summary>

| | | |
|---|---|---|
| ![](figures/hic_maps/before_yahs/hic_map_before_02_zoom.png) | ![](figures/hic_maps/before_yahs/hic_map_before_03.png) | ![](figures/hic_maps/before_yahs/hic_map_before_04.png) |
| ![](figures/hic_maps/before_yahs/hic_map_before_05.png) | ![](figures/hic_maps/before_yahs/hic_map_before_06.png) | ![](figures/hic_maps/before_yahs/hic_map_before_11.png) |
| ![](figures/hic_maps/before_yahs/hic_map_before_12.png) | ![](figures/hic_maps/before_yahs/hic_map_before_13.png) | ![](figures/hic_maps/before_yahs/hic_map_before_14.png) |
| ![](figures/hic_maps/before_yahs/hic_map_before_15.png) | ![](figures/hic_maps/before_yahs/hic_map_before_16.png) | ![](figures/hic_maps/before_yahs/hic_map_before_17.png) |

</details>

---

### After YaHS Scaffolding (Step 14)

After YaHS, the contact map shows **16–17 clear triangular blocks** — each block = one *S. cerevisiae* chromosome. This confirms successful chromosome-level assembly. ✅

![After YaHS — full view](figures/hic_maps/after_yahs/hic_map_after_01_full.png)

| Detail view 1 | Detail view 2 |
|---|---|
| ![](figures/hic_maps/after_yahs/hic_map_after_08.png) | ![](figures/hic_maps/after_yahs/hic_map_after_09.png) |

| Detail view 3 | Detail view 4 |
|---|---|
| ![](figures/hic_maps/after_yahs/hic_map_after_12.png) | ![](figures/hic_maps/after_yahs/hic_map_after_14.png) |

<details>
<summary>View all 14 post-scaffolding contact maps</summary>

| | | |
|---|---|---|
| ![](figures/hic_maps/after_yahs/hic_map_after_02.png) | ![](figures/hic_maps/after_yahs/hic_map_after_03.png) | ![](figures/hic_maps/after_yahs/hic_map_after_04.png) |
| ![](figures/hic_maps/after_yahs/hic_map_after_05.png) | ![](figures/hic_maps/after_yahs/hic_map_after_06.png) | ![](figures/hic_maps/after_yahs/hic_map_after_07.png) |
| ![](figures/hic_maps/after_yahs/hic_map_after_10.png) | ![](figures/hic_maps/after_yahs/hic_map_after_11.png) | ![](figures/hic_maps/after_yahs/hic_map_after_13.png) |

</details>

---

### Before vs After Comparison

| Before YaHS (Step 11) | After YaHS (Step 14) |
|---|---|
| ![Before](figures/hic_maps/before_yahs/hic_map_before_01_full.png) | ![After](figures/hic_maps/after_yahs/hic_map_after_01_full.png) |
| Thin diagonal — contigs not yet in chromosome order | 16 clear blocks — each = one chromosome ✅ |

---

## Galaxy Workflow Screenshots

### Galaxy History — After Meryl Step

![Galaxy history after Meryl](figures/galaxy_screenshots/galaxy_history_after_meryl.png)

> History panel showing `HiFi_collection (trimmed)` successfully created and Meryl jobs queued (3 jobs running in parallel on the 3 FASTA files).

### HiFi Collection Successfully Created

![Collection created](figures/galaxy_screenshots/galaxy_history_collection_created.png)

> Galaxy History showing `HiFi_collection` as a Dataset List containing 3 FASTA files — ready to be used as collection input for Cutadapt and Meryl.

### Galaxy Collection Builder Interface

![Collection builder](figures/galaxy_screenshots/galaxy_collection_builder.png)

> The Galaxy collection builder showing the "Flat List" option selected for grouping the 3 HiFi FASTA files into one collection item.

---

## Repository Structure

```
vgp-genome-assembly/
│
├── README.md                           ← This file (all figures embedded)
│
├── docs/
│   ├── 01_pipeline_overview.md         ← Why each phase exists, biology explained
│   ├── 02_input_output_table.md        ← Every step: input → tool → output
│   ├── 03_tool_parameters.md           ← Exact settings for every tool
│   └── 04_results_interpretation.md    ← How to read BUSCO, Merqury, contact maps
│
├── results/
│   ├── genomescope_summary.txt         ← GenomeScope2 output
│   ├── assembly_stats.txt              ← Gfastats contig statistics
│   ├── busco_hap1_summary.txt          ← BUSCO (Hap1: 95.9%)
│   ├── busco_hap2_summary.txt          ← BUSCO (Hap2: 89.0%)
│   └── merqury_completeness.txt        ← Merqury (99.9999%)
│
├── figures/
│   ├── genomescope/                    ← 4 GenomeScope2 profile plots
│   ├── busco/                          ← 2 BUSCO bar charts
│   ├── merqury/                        ← 12 Merqury spectrum plots
│   ├── hic_maps/
│   │   ├── before_yahs/                ← 17 pre-scaffolding contact maps
│   │   └── after_yahs/                 ← 14 post-scaffolding contact maps
│   └── galaxy_screenshots/             ← 3 Galaxy workflow screenshots
│
└── scripts/
    └── upload_urls.txt                 ← All Zenodo URLs for input data
```

---

## References

1. **Rhie A. et al.** (2021). Towards complete and error-free genome assemblies of all vertebrate species. *Nature*, 592, 737–746. https://doi.org/10.1038/s41586-021-03451-0
2. **Cheng H. et al.** (2021). Haplotype-resolved de novo assembly using phased assembly graphs with hifiasm. *Nature Methods*, 18, 170–175. https://doi.org/10.1038/s41592-020-01056-5
3. **Rhie A. et al.** (2020). Merqury: reference-free quality, completeness, and phasing assessment for genome assemblies. *Genome Biology*, 21, 245. https://doi.org/10.1186/s13059-020-02134-9
4. **Zhou C., McCarthy S.A., Durbin R.** (2022). YaHS: yet another Hi-C scaffolding tool. *Bioinformatics*, 39, btac808. https://doi.org/10.1093/bioinformatics/btac808
5. **Simão F.A. et al.** (2015). BUSCO: assessing genome assembly and annotation completeness. *Bioinformatics*, 31, 3210–3212. https://doi.org/10.1093/bioinformatics/btv351
6. **Formenti G. et al.** (2022). Gfastats: conversion, evaluation and manipulation of genome sequences. *Bioinformatics*, 38, 4214–4216. https://doi.org/10.1093/bioinformatics/btac460
7. **Ranallo-Benavidez T.R. et al.** (2020). GenomeScope 2.0 and Smudgeplots. *Nature Communications*, 11, 1432. https://doi.org/10.1038/s41467-020-14998-3
8. **Galaxy Training Network.** VGP genome assembly tutorial. https://training.galaxyproject.org/training-material/topics/assembly/tutorials/vgp_genome_assembly/tutorial.html

---

