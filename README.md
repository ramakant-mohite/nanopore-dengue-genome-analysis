# Nanopore Dengue Genome Analysis Pipeline

This repository documents a bioinformatics workflow for analyzing Dengue virus whole genome sequencing data generated using Oxford Nanopore sequencing technology.

The pipeline processes raw sequencing reads and generates a consensus viral genome.

---

## Workflow Overview

1. Basecalling  
2. Demultiplexing  
3. Read Mapping  
4. Primer Trimming  
5. Coverage Masking  
6. Variant Calling  
7. Consensus Genome Generation  

---

## 1. Basecalling

Basecalling converts raw electrical signals produced by the Nanopore sequencer into nucleotide sequences.

**Tool used:**  
Guppy

**Example command**

```
guppy_basecaller -i input_fast5 -s Basecalling -c dna_r9.4.1_450bps_fast.cfg -x cuda:0
```

---

## 2. Demultiplexing

Demultiplexing separates reads from multiple samples based on barcode sequences.

**Tool used:**  
Guppy

**Example command**

```
guppy_barcoder -i Basecalling/pass -s Demultiplexing --barcode_kits EXP-NBD104
```

---

## 3. Read Mapping

Sequencing reads are aligned to the Dengue virus reference genome.

**Tools used**

- Minimap2  
- Samtools  

**Example commands**

```
minimap2 -a -x map-ont denv2.reference.fasta barcode13.fastq | samtools view -bS -F 4 - | samtools sort -o barcode13.sorted.bam
samtools index barcode13.sorted.bam
```

---

## 4. Primer Trimming

Primer sequences used during amplification are removed before variant analysis.

**Tool used**

ARTIC pipeline (align_trim)

**Example command**

```
align_trim --normalise 200 denv2.scheme.bed --remove-incorrect-pairs --report barcode13.alignreport.txt < barcode13.sorted.bam | samtools sort -o barcode13.trimmed.rg.sorted.bam
```

---

## 5. Coverage Masking

Low coverage regions are masked to prevent incorrect variant calls.

**Tool used**

ARTIC mask

**Example command**

```
artic_make_depth_mask denv2.reference.fasta barcode13.primertrimmed.rg.sorted.bam barcode13.coverage_mask.txt
```

---

## 6. Variant Calling

Genetic variants are identified relative to the Dengue reference genome.

**Tool used**

Clair3

**Example command**

```
run_clair3.sh \
--bam_fn barcode13.trimmed.rg.sorted.bam \
--ref_fn denv2.reference.fasta \
--threads 40 \
--platform ont \
--model_path models/r941_prom_hac_g238 \
--output barcode13
```

---

## 7. Variant Filtering

Variants are filtered using genotype quality and sequencing depth.

**Tool used**

bcftools

**Example commands**

```
bcftools view --include 'MIN(FMT/DP)>20 & MIN(FMT/GQ)>3' barcode13-merge_output.vcf > barcode13.pass.vcf
```
```
bcftools view --include 'MIN(FMT/DP)<=20 | MIN(FMT/GQ)<=3' barcode13-merge_output.vcf > barcode13.fail.vcf
```

---

## 8. Consensus Genome Generation

A final consensus genome sequence is generated for each sample.

**Tool used**

bcftools

**Example command**

```
bcftools consensus -f barcode13.preconsensus.fasta barcode13.pass.vcf
-m barcode13.coverage_mask.txt
-o barcode13.consensus.fasta
```

---

## Tools Used

- Guppy  
- Minimap2  
- Samtools  
- ARTIC pipeline  
- Clair3  
- bcftools  

---
## Related Publication

This workflow contributed to genomic analysis performed in the following study:

Ravi V., Khare K., Mohite R., Mishra P., Halder S., Shukla R., Liu C.S.C., Yadav A., Soni J., Kanika K., Chaudhary K., Neha N., Tarai B., Budhiraja S., Khosla P., Sethi T., Imran M., Pandey R.  
**Genomic hotspots in the DENV-2 serotype (E, NS4B, and NS5 genes) are associated with dengue disease severity in the endemic region of India.**  
*PLOS Neglected Tropical Diseases* (2025)

https://doi.org/10.1371/journal.pntd.0013034

---

## Applications

This workflow can be used for:

- Dengue virus genomic surveillance  
- Viral genome assembly  
- Variant detection  
- Molecular epidemiology studies
