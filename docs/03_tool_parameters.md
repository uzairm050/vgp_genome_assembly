# Exact Tool Parameters

All settings used for every tool in the VGP pipeline. Copy-paste ready for reproducing this analysis.

---

## Step 1 — Cutadapt

```
Single-end or Paired-end reads: Single-end
FASTQ/A file: HiFi_collection (collection)

Read 1 Adapters → 5' or 3' (Anywhere) Adapters:
  Adapter 1:
    Name: First adapter
    Sequence: ATCTCTCTCAACAACAACAACGGAGGAGGAGGAAAAGAGAGAGAT
  Adapter 2:
    Name: Second adapter
    Sequence: ATCTCTCTCTTTTCCTCCTCCTCCGTTGTTGTTGTTGAGAGAGAT

Adapter Options:
  Maximum error rate: 0.1
  Minimum overlap length: 35
  Look for adapters in reverse complement: Yes

Filter Options:
  Discard Trimmed Reads: Yes
```

---

## Step 2a — Meryl (Count)

```
Operation type selector: Count operations
Count operations: Count: count the occurrences of canonical k-mers
Input sequences: HiFi_collection (trimmed)  [collection input]
k-mer size selector: Set a k-mer size
k-mer size: 31
```

## Step 2b — Meryl (Union-sum)

```
Operation type selector: Operations on sets of k-mers
Operations on sets of k-mers: Union-sum: return k-mers that occur in any input,
                               set the count to the sum of the counts
Input meryldb: meryldb  [collection from 2a]
```

## Step 2c — Meryl (Histogram)

```
Operation type selector: Generate histogram dataset
Input meryldb: Merged meryldb
```

---

## Step 3 — GenomeScope2

```
Input histogram file: meryldb histogram
Ploidy for model to use: 2
k-mer length used to calculate k-mer spectra: 31

Output options:
  Summary of the analysis: Yes

Advanced options:
  Create testing.tsv file with model parameters: Yes
```

---

## Step 4 — Hifiasm

```
Assembly mode: Standard
Input reads: HiFi_collection (trimmed)  [collection input]

Options for Hi-C partition: Specify
  Hi-C R1 reads: Hi-C_dataset_F
  Hi-C R2 reads: Hi-C_dataset_R

All other settings: default
```

---

## Step 5a — Gfastats (GFA → FASTA)

```
Input GFA file: Hap1 contigs graph + Hap2 contigs graph  [multiple datasets]
Tool mode: Genome assembly manipulation
Output format: FASTA
Generates the initial set of paths: Yes
```

## Step 5b — Gfastats (Statistics)

```
Input file: Hap1 contigs graph + Hap2 contigs graph  [multiple datasets]
Tool mode: Summary statistics generation
Expected genome size: 11747160
Thousands separator in output: No
```

## Step 5c — Column Join + Search in Textfiles

```
Column join:
  Input file: Hap1 stats + Hap2 stats  [both files]
  All other settings: default

Search in textfiles:
  Input file: gfastats on hap1 and hap2 (full)
  that: Don't Match
  Type of regex: Basic
  Regular Expression: scaffold
  Match type: case insensitive
```

---

## Step 6 — BUSCO

```
Sequences to analyze: Hap1 contigs FASTA + Hap2 contigs FASTA  [multiple datasets]
Lineage data source: Use cached lineage data
Cached database with lineage: Busco v5 Lineage Datasets (2023-05-02)
Mode: Genome assemblies (DNA)
Use Augustus instead of Metaeuk: Use Metaeuk
Auto-detect or select lineage?: Select lineage
Lineage: saccharomycetes_odb10
Which outputs should be generated: short summary text + summary image
```

---

## Step 7 — Merqury

```
Evaluation mode: Default mode
k-mer counts database: Merged meryldb
Number of assemblies: Two assemblies
First genome assembly: Hap1 contigs FASTA
Second genome assembly: Hap2 contigs FASTA
```

---

## Step 8b — Bionano Hybrid Scaffold

```
NGS FASTA: Hap1 contigs FASTA
BioNano CMAP: Bionano_dataset
Configuration mode: VGP mode
Genome maps conflict filter: Cut contig at conflict
Sequences conflict filter: Cut contig at conflict
```

## Step 8c — Concatenate Datasets

```
Concatenate Dataset: NGScontigs scaffold NCBI trimmed
Insert Dataset: NGScontigs not scaffolded trimmed
```

---

## Steps 9 & 10 — BWA-MEM (run twice)

```
Will you select a reference genome from your history: Use a genome from history and build index
Reference sequence: Hap1 assembly bionano
Single or Paired-end reads: Single
Select fastq dataset: Hi-C_dataset_F  [Step 9]  or  Hi-C_dataset_R  [Step 10]
Set read groups information?: Do not set
Select analysis mode: 1. Simple Illumina mode
BAM sorting mode: Sort by read names (i.e., the QNAME field)
```

## Step 10b — Filter and Merge

```
First set of reads: BAM forward
Second set of reads: BAM reverse
```

---

## Step 11a — PretextMap

```
Input dataset in SAM or BAM format: BAM Hi-C reads
Sort by: Don't sort
```

## Step 11b — Pretext Snapshot

```
Input Pretext map file: PretextMap output
Output image format: png
Show grid?: Yes
```

---

## Step 12 — YaHS

```
Input contig sequences: Hap1 assembly bionano
Alignment file of Hi-C reads to contigs: BAM Hi-C reads
Restriction enzyme used in Hi-C experiment: Enter a specific sequence
Restriction enzyme sequence(s): CTTAAG
```

---

## Steps 13 — BWA-MEM ×2 + Filter and Merge (repeat)

Same as Steps 9, 10, 10b but:
```
Reference sequence: YaHS Scaffolds FASTA  ← only change
```
Outputs renamed: BAM forward YaHS, BAM reverse YaHS, BAM Hi-C reads YaHS

---

## Step 14a — PretextMap (final)

```
Input dataset in SAM or BAM format: BAM Hi-C reads YaHS
Sort by: Don't sort
```
Rename output: PretextMap output YaHS

## Step 14b — Pretext Snapshot (final)

```
Input Pretext map file: PretextMap output YaHS
Output image format: png
Show grid?: Yes
```
