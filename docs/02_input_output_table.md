# Input → Tool → Output Reference Table

Every step of the VGP pipeline: exactly what goes in, which tool runs, and what comes out.

---

## Phase 1 — Prepare the Data

| Step | Input | Tool | Output |
|---|---|---|---|
| 1 | `HiFi_collection` (3 × FASTA) | **Cutadapt** — adapter trimming | `HiFi_collection (trimmed)` |
| 2a | `HiFi_collection (trimmed)` | **Meryl** — count k-mers (k=31) | `meryldb` (collection of 3) |
| 2b | `meryldb` collection | **Meryl** — union-sum merge | `Merged meryldb` |
| 2c | `Merged meryldb` | **Meryl** — generate histogram | `meryldb histogram` |
| 3 | `meryldb histogram` | **GenomeScope2** — genome profiling | Profile plots + `summary.txt` |

---

## Phase 2 — Assemble the Genome

| Step | Input | Tool | Output |
|---|---|---|---|
| 4 | `HiFi_collection (trimmed)` + `Hi-C_dataset_F` + `Hi-C_dataset_R` | **Hifiasm** — Hi-C phased mode | `Hap1 contigs graph (GFA)` + `Hap2 contigs graph (GFA)` |
| 5a | `Hap1 contigs graph` + `Hap2 contigs graph` | **Gfastats** — manipulation mode (GFA→FASTA) | `Hap1 contigs FASTA` + `Hap2 contigs FASTA` |
| 5b | `Hap1 contigs graph` + `Hap2 contigs graph` | **Gfastats** — summary statistics | `Hap1 stats` + `Hap2 stats` |
| 5c | `Hap1 stats` + `Hap2 stats` | **Column join** + **Search in textfiles** | `gfastats on hap1 and hap2 contigs` |

---

## Phase 3 — Quality Control

| Step | Input | Tool | Output |
|---|---|---|---|
| 6 | `Hap1 contigs FASTA` + `Hap2 contigs FASTA` | **BUSCO** — Saccharomycetes lineage | `BUSCO hap1 summary` + `BUSCO hap2 summary` + images |
| 7 | `Merged meryldb` + `Hap1 contigs FASTA` + `Hap2 contigs FASTA` | **Merqury** — default mode | Completeness stats + CN plots + ASM plots |

---

## Phase 4 — Scaffold to Chromosome Level

| Step | Input | Tool | Output |
|---|---|---|---|
| 8a | Zenodo URL | **Galaxy Upload** | `Bionano_dataset (.cmap)` |
| 8b | `Hap1 contigs FASTA` + `Bionano_dataset` | **Bionano Hybrid Scaffold** | `NGScontigs scaffold NCBI trimmed` + `NGScontigs not scaffolded trimmed` |
| 8c | Both Bionano outputs | **Concatenate datasets** | `Hap1 assembly bionano` |
| 8d | `Hap1 assembly bionano` | **Gfastats** — statistics | `Bionano stats` |
| 9 | `Hap1 assembly bionano` + `Hi-C_dataset_F` (Single mode) | **BWA-MEM** — map forward reads | `BAM forward` |
| 10 | `Hap1 assembly bionano` + `Hi-C_dataset_R` (Single mode) | **BWA-MEM** — map reverse reads | `BAM reverse` |
| 10b | `BAM forward` + `BAM reverse` | **Filter and merge** (Arima Genomics) | `BAM Hi-C reads` |
| 11a | `BAM Hi-C reads` | **PretextMap** — sort=Don't sort | `PretextMap output` |
| 11b | `PretextMap output` | **Pretext Snapshot** — format=PNG, grid=Yes | `Initial Hi-C contact map (PNG)` |
| 12 | `Hap1 assembly bionano` + `BAM Hi-C reads` + enzyme `CTTAAG` | **YaHS** — Hi-C scaffolding | `YaHS Scaffolds FASTA` |
| 13 | `YaHS Scaffolds FASTA` + `Hi-C_dataset_F` + `Hi-C_dataset_R` | **BWA-MEM ×2** + **Filter and merge** | `BAM Hi-C reads YaHS` |
| 14a | `BAM Hi-C reads YaHS` | **PretextMap** | `PretextMap output YaHS` |
| 14b | `PretextMap output YaHS` | **Pretext Snapshot** | `Final Hi-C contact map (PNG)` |

---

## Data Flow Summary

```
HiFi reads ──► Cutadapt ──► Meryl ──► GenomeScope2
                  │
                  └──► Hifiasm ◄── Hi-C reads
                           │
                           ▼
                       Gfastats ──► BUSCO
                           │           │
                           └──► Merqury (+ Meryldb)
                           │
                           └──► Bionano Scaffold ◄── bionano.cmap
                                       │
                                       ▼
                                  BWA-MEM ◄── Hi-C reads
                                       │
                                       ▼
                                  Filter & Merge
                                       │
                                       ▼
                                  PretextMap ──► Initial map
                                       │
                                       ▼
                                     YaHS
                                       │
                                  Remap Hi-C ──► Final map
                                       │
                                       ▼
                              CHROMOSOME-LEVEL ASSEMBLY
```
