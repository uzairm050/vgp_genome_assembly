# Results Interpretation Guide

How to read and understand every output produced by the VGP pipeline.

---

## GenomeScope2 Output

### The K-mer Profile Plot

The GenomeScope2 plot shows k-mer frequency (y-axis) vs coverage (x-axis).

```
What each feature means:

Orange peak (near 0x)  → Sequencing errors (k-mers appearing only once)
First peak (~25x)      → Heterozygous k-mers (present in one chromosome copy)
Second peak (~50x)     → Homozygous k-mers (present in both chromosome copies)
Black line             → Mathematical model fit to the data
```

**Our results:**
- Two clear peaks at 25x and 50x confirm a diploid genome ✅
- Model fit 96.5% → results are trustworthy ✅
- Very small error peak → high quality HiFi reads ✅

### Summary Statistics

```
Genome Haploid Length: 11,743,432 bp  → Use maximum value (11,747,352) for downstream tools
Heterozygosity:        0.58%          → Very low; genome is mostly homozygous
Model Fit:             96.5%          → Excellent model fit
Read Error Rate:       0.00094%       → Near-zero error rate in HiFi reads
```

---

## Gfastats Assembly Statistics

### Key Metrics Explained

| Metric | What it means | Our value |
|---|---|---|
| **# contigs** | Number of assembled pieces | 16 (Hap1), 17 (Hap2) |
| **Total contig length** | Sum of all contig lengths | ~11.3 Mb (Hap1) |
| **Largest contig** | Length of the longest contig | ~1.53 Mb |
| **Contig N50** | 50% of assembly is in contigs this size or larger | ~922 kb |
| **Contig auN** | Area under the Nx curve — more sensitive than N50 | — |

### How to Calculate N50

```
1. Sort contigs: longest → shortest
2. Add lengths from top until you reach 50% of total assembly size
3. The length of the contig at that point = N50

Example: Total = 100 Mb → stop at 50 Mb → contig at boundary = N50
```

---

## BUSCO Results

### Bar Chart Categories

| Color | Category | Meaning | Good value |
|---|---|---|---|
| Light blue | Complete single-copy (S) | Gene found once — perfect | >90% |
| Dark blue | Complete duplicated (D) | Gene found twice — possible false dup | <5% |
| Yellow | Fragmented (F) | Gene partially found | <5% |
| Red | Missing (M) | Gene not found | <5% |

### Our Results

```
Hap1: C:95.9% [S:94.1%, D:1.8%], F:2.7%, M:1.4%  → EXCELLENT
Hap2: C:89.0% [S:87.6%, D:1.4%], F:2.5%, M:8.5%  → GOOD

Low duplication (1.8% / 1.4%) confirms NO false duplications in assembly.
```

### Why Hap2 has More Missing Genes

Hap2 (alternate haplotype) is expected to be slightly less complete than Hap1 because:
- Homozygous regions are represented in Hap1 and referenced from Hap2
- The alternate assembly naturally covers fewer unique regions

---

## Merqury Plots

### CN Spectrum Plot (spectra-cn)

Shows k-mer copy numbers colored by how many times they appear in the assembly.

```
Color coding:
  Grey  = read-only k-mers (in reads but NOT in assembly) → sequencing errors or alternate alleles
  Red   = 1-copy k-mers in assembly (heterozygous regions)
  Blue  = 2-copy k-mers in assembly (homozygous regions)

Good assembly: large red peak at ~25x + large blue peak at ~50x
Bad assembly:  large grey peak at ~25x (many k-mers missing from assembly)
```

### ASM Spectrum Plot (spectra-asm) — Most Important!

Shows which k-mers belong to which assembly.

```
Color coding:
  Grey  = read-only (not in either assembly)
  Red   = Hap1-only k-mers (heterozygous variants in Hap1)
  Blue  = Hap2-only k-mers (heterozygous variants in Hap2)
  Green = Shared k-mers (homozygous regions in both assemblies)

IDEAL pattern:
  Large GREEN peak at ~50x  → homozygous regions correctly shared
  RED peak at ~25x           → Hap1-specific variants
  BLUE peak at ~25x          → Hap2-specific variants (similar size to red)
  Very small grey peak        → very few missing k-mers
```

**Our result:** Large green peak at 50x + balanced red and blue peaks at 25x = **excellent phasing** ✅

### Completeness Statistics

```
Assembly    k-mers found    Total k-mers    Completeness
Hap1        10,792,811      13,010,260      82.96%
Hap2        11,611,483      13,010,260      89.25%
Both        13,010,244      13,010,260      99.9999%   ← near perfect!
```

> The combined completeness (99.9999%) means virtually every k-mer in the reads is represented somewhere in one of the two assemblies.

---

## Hi-C Contact Maps

### How to Read a Contact Map

```
Axes:     Both X and Y show the genome in linear order
Dot:      A dot at position (A, B) means regions A and B physically contacted in the cell
Color:    Darker/warmer = more contacts (higher interaction frequency)
Diagonal: Represents self-contacts (region A with itself) — always most intense
Blocks:   A triangular block on the diagonal = one chromosome
```

### Interpreting Quality

| Pattern | Meaning |
|---|---|
| Clean triangular blocks on diagonal | Good — each block = one chromosome |
| All contacts near diagonal | Good — contigs are in correct order |
| Off-diagonal signal within a block | Normal chromatin loops within chromosome |
| Off-diagonal signal between blocks | Problem — possible misassembly |
| Inverted diagonal segment | Inversion — contig is in wrong orientation |

### Before vs After YaHS

```
BEFORE scaffolding (Step 11):
  - Thin diagonal line
  - Many small segments
  - No clear chromosome blocks
  - Confirms contigs are internally correct but unassembled

AFTER scaffolding (Step 14):
  - 16-17 clear triangular blocks
  - Each block = one chromosome
  - Clean boundaries between chromosomes
  - Confirms successful chromosome-level assembly ✅
```

---

## Summary of All QC Checkpoints

| Step | Tool | What passes | What fails |
|---|---|---|---|
| Step 3 | GenomeScope2 | Two clear peaks + >90% model fit | Noisy histogram, poor fit |
| Step 5 | Gfastats | N50 >500 kb, contig count near chromosome number | Very fragmented (N50 <10 kb) |
| Step 6 | BUSCO | >90% complete, <5% duplicated | <80% complete or >10% duplicated |
| Step 7 | Merqury | >99% combined completeness | <95% combined completeness |
| Step 11 | Initial contact map | Clean diagonal confirms good contigs | Messy off-diagonal patterns |
| Step 14 | Final contact map | 16 clear chromosome blocks | Unclear blocks, misassembly patterns |
