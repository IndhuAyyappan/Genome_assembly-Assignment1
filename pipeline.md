# Pipeline (Assignment 1): ONT reads - Flye assembly - QUAST - read-based variant calling (Clair3) - IGV

This document has the exact workflow used for **Salmonella enterica** ONT R10 reads (`SRR32410565.fastq`).
The assignment goal is to:
1) Assemble a genome (Flye)  
2) Compare assembly to a reference genome (QUAST + assembly to ref alignment)  
3) Call variants vs the reference genome (Clair3; read-based)  
4) Visualize differences in IGV (BAM + VCF on reference coordinates)

---
### MAIN workflow used in Discussion 
Raw ONT reads  
**Flye HQ assembly** (final assembly)  
**QUAST vs reference** + **assembly to reference BAM** (structure)  

AND  

Raw ONT reads  
**minimap2 map-ont to reference**  
**Clair3 variants vs reference**  
**IGV** (reference + BAM + VCF)

Only these steps and outputs are used for the final biological interpretation.

###  Exploratory / extra outputs (not used for the main Discussion)
These analyses were performed to validate assumptions or explore alternative strategies, but are not used in final figures or interpretation:

- Flye `--nano-raw` baseline assembly + QUAST (exploratory)
- Clair3 rerun with `--no_phasing_for_fa` (haploid check; exploratory)
- Clair3 runs where assemblies (Flye HQ / Medaka) were used as references (polishing comparison; exploratory)
- Medaka polishing + QUAST comparisons

All exploratory outputs are clearly separated in folder structure.

---

## Inputs
- Raw ONT reads: `data/raw/SRR32410565.fastq`
- Reference genome: `data/reference/GCF_000006945.2_ASM694v2_genomic.fna`

## Main output folders
- QC: `results/qc/`
- Assemblies: `results/flye/`, `results/flye_hq/`
- Alignments: `results/alignment/`
- QUAST: `results/quast/`
- Polishing: `results/polishing/medaka_flye_hq/`
- Variant calling: `results/variant_calling/`

---


## 0) Raw read QC (exploratory)

### 0.1 FastQC

fastqc data/raw/SRR32410565.fastq


### 0.2 Read length distribution
Purpose: To assess read length distribution of raw ONT reads before assembly.

mkdir -p results/qc
readlength.sh in=data/raw/SRR32410565.fastq > results/qc/readlength.txt


## 1) Genome assembly (Flye)
### 1.1 Flye baseline assembly (exploratory)
Purpose: quick baseline assembly using --nano-raw.

docker run --rm \
  --platform linux/amd64 \
  -v "$PWD":/work \
  staphb/flye \
  flye \
    --nano-raw /work/data/raw/SRR32410565.fastq \
    --out-dir /work/results/flye \
    --threads 6

Output:
	•	results/flye/assembly.fasta

### 1.2 Flye HQ assembly (FINAL assembly)
Purpose: produce higher-quality assembly using --nano-hq.

docker run --rm \
  --platform linux/amd64 \
  -v "$PWD":/work \
  staphb/flye \
  flye \
    --nano-hq /work/data/raw/SRR32410565.fastq \
    --genome-size 4.8m \
    --out-dir /work/results/flye_hq \
    --threads 6
Output:
	•	results/flye_hq/assembly.fasta


## 2) Assembly evaluation against the reference
Purpose: To evaluate contiguity and reference alignment statistics for the initial assembly.
### 2.1 QUAST: baseline Flye vs reference (exploratory)
mkdir -p results/quast

docker run --rm \
  --platform linux/amd64 \
  -v "$PWD":/work \
  quay.io/biocontainers/quast:5.2.0--py310pl5321hc8f18ef_2 \
  quast.py /work/results/flye/assembly.fasta \
    -r /work/data/reference/GCF_000006945.2_ASM694v2_genomic.fna \
    -o /work/results/quast/flye_vs_ref

### 2.2 QUAST: Flye HQ vs reference (MAIN)

mkdir -p results/quast

docker run --rm \
  --platform linux/amd64 \
  -v "$PWD":/work \
  quay.io/biocontainers/quast:5.2.0--py310pl5321hc8f18ef_2 \
  quast.py /work/results/flye_hq/assembly.fasta \
    -r /work/data/reference/GCF_000006945.2_ASM694v2_genomic.fna \
    -o /work/results/quast/flye_hq_vs_ref

Outputs:
	•	results/quast/flye_hq_vs_ref/report.html
	•	results/quast/flye_hq_vs_ref/report.txt

## 3) Assembly to reference alignment (for IGV structural inspection)
Purpose: To align the initial Flye assembly to the reference genome to assess large scale structural consistency.
### 3.1  Flye HQ assembly to reference (MAIN track in IGV)
mkdir -p results/alignment

minimap2 -ax asm5 \
  data/reference/GCF_000006945.2_ASM694v2_genomic.fna \
  results/flye_hq/assembly.fasta \
  > results/alignment/flye_hq_to_ref.sam

samtools view -bS results/alignment/flye_hq_to_ref.sam > results/alignment/flye_hq_to_ref.bam
samtools sort results/alignment/flye_hq_to_ref.bam -o results/alignment/flye_hq_to_ref.sorted.bam
samtools index results/alignment/flye_hq_to_ref.sorted.bam

Outputs:
	•	results/alignment/flye_hq_to_ref.sorted.bam
	•	results/alignment/flye_hq_to_ref.sorted.bam.bai

## 4) Read-based alignment to reference (for variant calling + IGV)
Purpose: To align raw ONT reads directly to the reference genome for read based variant calling.
### 4.1 Raw reads to reference (MAIN BAM for Clair3 + IGV)
mkdir -p results/alignment

minimap2 -ax map-ont -t 6 \
  data/reference/GCF_000006945.2_ASM694v2_genomic.fna \
  data/raw/SRR32410565.fastq \
  > results/alignment/raw_to_ref.sam

samtools view -bS results/alignment/raw_to_ref.sam > results/alignment/raw_to_ref.bam
samtools sort results/alignment/raw_to_ref.bam -o results/alignment/raw_to_ref.sorted.bam
samtools index results/alignment/raw_to_ref.sorted.bam

Outputs:
	•	results/alignment/raw_to_ref.sorted.bam
	•	results/alignment/raw_to_ref.sorted.bam.bai

## 5) Variant calling (Clair3)
Purpose: To call variants using Clair3.
### 5.1 Clair3: raw reads to reference (MAIN results)
Notes:
	•	Salmonella enterica is haploid
	•	Clair3 reports genotypes using diploid VCF notation (e.g., 1/1) by convention
	•	Clair3 outputs one VCF per reference contig, automatically generated in tmp/merge_output/

mkdir -p results/variant_calling/clair3_vc_raw_to_ref

docker run --rm \
  -v "$PWD":/data \
  hkubal/clair3:latest \
  /opt/bin/run_clair3.sh \
  --bam_fn=/data/results/alignment/raw_to_ref.sorted.bam \
  --ref_fn=/data/data/reference/GCF_000006945.2_ASM694v2_genomic.fna \
  --threads=6 \
  --platform="ont" \
  --model_path="/opt/models/r1041_e82_400bps_sup_v500" \
  --output=/data/results/variant_calling/clair3_vc_raw_to_ref

Outputs used in IGV:
	•	results/variant_calling/clair3_vc_raw_to_ref/tmp/merge_output/merge_NC_003197.2.vcf  (main chromosome)
	•	results/variant_calling/clair3_vc_raw_to_ref/tmp/merge_output/merge_NC_003277.2.vcf  (secondary replicon / plasmid)


### 5.2 Clair3 rerun: raw reads to reference with haploid/no-phasing flag (exploratory)
Purpose: sanity-check calling behavior in a haploid organism.

mkdir -p results/variant_calling/clair3_vc_raw_to_ref_nophase

docker run --rm \
  -v "$PWD":/data \
  hkubal/clair3:latest \
  /opt/bin/run_clair3.sh \
  --bam_fn=/data/results/alignment/raw_to_ref.sorted.bam \
  --ref_fn=/data/data/reference/GCF_000006945.2_ASM694v2_genomic.fna \
  --threads=6 \
  --platform="ont" \
  --model_path="/opt/models/r1041_e82_400bps_sup_v500" \
  --no_phasing_for_fa \
  --output=/data/results/variant_calling/clair3_vc_raw_to_ref_nophase


## 6) Medaka polishing (exploratory comparison)
### 6.1 Medaka polish Flye HQ assembly (exploratory)

mkdir -p results/polishing/medaka_flye_hq

docker run --rm \
  -v "$PWD":/data \
  ontresearch/medaka \
  medaka_consensus \
  -i /data/data/raw/SRR32410565.fastq \
  -d /data/results/flye_hq/assembly.fasta \
  -o /data/results/polishing/medaka_flye_hq \
  -t 6 \
  -m r1041_e82_400bps_sup_v5.2.0

Output:
	•	results/polishing/medaka_flye_hq/consensus.fasta

### 6.2 QUAST: Medaka consensus vs reference (exploratory)

docker run --rm \
  --platform linux/amd64 \
  -v "$PWD":/work \
  quay.io/biocontainers/quast:5.2.0--py310pl5321hc8f18ef_2 \
  quast.py /work/results/polishing/medaka_flye_hq/consensus.fasta \
    -r /work/data/reference/GCF_000006945.2_ASM694v2_genomic.fna \
    -o /work/results/quast/medaka_flye_hq_vs_ref

### 6.3 QUAST: Flye HQ vs Medaka vs reference (exploratory)

docker run --rm \
  --platform linux/amd64 \
  -v "$PWD":/work \
  quay.io/biocontainers/quast:5.2.0--py310pl5321hc8f18ef_2 \
  quast.py \
    /work/results/flye_hq/assembly.fasta \
    /work/results/polishing/medaka_flye_hq/consensus.fasta \
    -r /work/data/reference/GCF_000006945.2_ASM694v2_genomic.fna \
    -o /work/results/quast/flye_hq_vs_medaka_vs_ref

## 7) IGV visualization (MAIN Result)

In IGV, load:
	1.	Reference genome: data/reference/GCF_000006945.2_ASM694v2_genomic.fna
	2.	BAM (reads to ref): results/alignment/raw_to_ref.sorted.bam
	3. VCF (variants vs reference):
   results/variant_calling/clair3_vc_raw_to_ref/merge_output.vcf.gz
	•	Clair3 per-contig VCFs (for screenshots):
	  •	results/variant_calling/clair3_vc_raw_to_ref/tmp/merge_output/merge_NC_003197.2.vcf
	  •	results/variant_calling/clair3_vc_raw_to_ref/tmp/merge_output/merge_NC_003277.2.vcf
	4.	BAM (FlyeHQ to ref): results/alignment/flye_hq_to_ref.sorted.bam

I used 3 representative variants on NC_003197.2 for figures:
	•	NC_003197.2:831 (SNP)
	•	NC_003197.2:15775 (SNP)
	•	NC_003197.2:19756 (indel)

Note:
Clair3 generates per-contig intermediate VCFs in tmp/merge_output/ (merge_NC_003197.2.vcf) was used for contig-specific IGV screenshots. The final merged variant callset used for interpretation is merge_output.vcf.gz.


