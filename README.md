# Genome Assembly of *Salmonella enterica*

## Introduction

Genome assembly is an important step in bioinformatics. It helps in reconstructing the genome of an organism from raw sequencing reads. In bacterial species such as *Salmonella enterica*, an accurate genome assembly is needed to study genome organization and to identify genetic differences between different isolates. This information is useful for downstream biological interpretation such as comparison between strains and outbreak related studies [1].

Even though bacterial genomes are smaller in size when compared to eukaryotic genomes, assembling them correctly is not easy. Repetitive regions, structural variation and sequencing errors can affect the reconstruction of a correct consensus genome [2].

Short-read sequencing technologies are commonly used for bacterial genome sequencing as they provide high base level accuracy. However, assemblies generated using short reads are often fragmented. This is mainly because short sequencing reads are not long enough to span repetitive regions or more complex genomic structures that are commonly present in bacterial genomes. When these regions are not spanned properly, the genome breaks into multiple contigs. As a result, certain regions of the genome such as plasmids or repeat rich regions may not be resolved correctly [3].

Long-read sequencing technologies such as Oxford Nanopore sequencing help reduce some of these issues. Since the reads are much longer, repetitive regions and larger structural variants can be spanned more easily, which often results in assemblies that are less fragmented and more complete [2].

At the same time, long-read sequencing also has its own limitations. Oxford Nanopore reads generally show higher per-read error rates compared to short reads, with insertion and deletion errors being commonly observed in homopolymer regions. These errors can affect base level accuracy in the final assembly if they are not taken into account during analysis [4].

Recent improvements in Oxford Nanopore sequencing, particularly advances in neural network basecalling models, have improved read accuracy for bacterial genomes [8]. This has made long-read-based genome assembly more practical for bacterial genomes. Even so, long-read assemblies may still contain residual errors. Hence, different genome assembly approaches involve considerations of contiguity, completeness and correctness. The assembly quality cannot be assessed using contiguity metrics alone [5].

## Proposed Methods

Oxford Nanopore R10 sequencing reads in FASTQ format will be used for this genome assembly. Before starting the assembly, the sequencing data will be checked to understand what it looks like. This includes looking at the read length distribution and the general sequencing quality. This step is important because Nanopore sequencing is known to have specific error patterns and these need to be considered before assembly [4].

Genome assembly will be done using Flye. Flye is a long-read assembler that is commonly used for bacterial genomes. It is suitable for this dataset because it is able to deal with repetitive regions and structural complexity, which are often present in bacterial genomes. Since the reads are long and error prone, a repeat graph–based assembler such as Flye is an appropriate choice [2].

After the assembly is generated, the assembled genome will be compared with a reference *Salmonella enterica* genome obtained from NCBI. The comparison will be done by aligning the assembly to the reference using minimap2. The alignment output will then be sorted and indexed using samtools so that it can be examined further. Visual inspection of the alignment will be carried out using IGV to look for structural variation and possible assembly related issues [7].Following alignment to the reference genome, variant calling will be performed to identify sequence level differences between the assembled genome and the reference. Single nucleotide variants and small insertions and deletions will be identified using Clair3, a variant caller designed for long-read sequencing data, including Oxford Nanopore reads [9]. The identified variants will be used to assess base level accuracy and to further characterize differences between the assembled genome and the reference strain.

## References

[1] Wick RR, Judd LM, Holt KE. *Assembling the perfect bacterial genome using Oxford Nanopore and Illumina sequencing*. PLOS Computational Biology, 2023.\
[2] Kolmogorov M, Yuan J, Lin Y, Pevzner PA. *Assembly of long, error-prone reads using repeat graphs*. Nature Biotechnology, 2019.\
[3] Liao Y et al. *Completing bacterial genome assemblies: strategy and performance comparisons*. Scientific Reports, 2015.\
[4] Delahaye C, Nicolas J. *Sequencing DNA with nanopores: Troubles and biases*. Bioinformatics, 2021.\
[5] Thrash A, Hoffmann F, Perkins A. *Toward a more holistic method of genome assembly assessment*. BMC Bioinformatics, 2020.\
[6] Li H. *Minimap2: pairwise alignment for nucleotide sequences*. Bioinformatics, 2018.\
[7] Thorvaldsdóttir H, Robinson JT, Mesirov JP. *Integrative Genomics Viewer (IGV): high-performance genomics data visualization*. Bioinformatics, 2013.\
[8] Wick RR, Judd LM, Holt KE. Performance of neural network basecalling tools for Oxford Nanopore sequencing. Genome Biology, 2019.                  
[9] Zheng Z, Li S, Su J, et al. Symphonizing pileup and full-alignment for deep learning–based long-read variant calling. Nature Computational Science, 2022.
