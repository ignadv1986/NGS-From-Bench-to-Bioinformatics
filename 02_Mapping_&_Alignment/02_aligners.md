# Types of Aligners

Choosing an aligner is one of the most consequential decisions in a bioinformatics pipeline. While the core "seed-and-extend" logic remains consistent, different algorithms have been optimized to handle specific biological phenomena, most notably the presence or absence of introns. An aligner used for DNA-seq (where the reference is a continuous string) will fail to map the majority of an RNA-seq library because it cannot "see" across the gaps left by spliced-out introns.

Beyond biology, aligners differ in their stringency, RAM requirements, speed, and output.

## STAR and HISAT2

These two aligners are the standard for RNA-seq analysis because they can handle intron-exon junctions. Since the cDNA is obtained from already mature mRNAs, where introns have already been removed, then these sequences will not be present in our library either. Therefore, if some reads fall between an exon-intron boundary (**splice junction**), it won't be possible to normally map them to the reference genome. STAR and HISAT2 allow to “jump” over the introns, marking that genomic region as a splice site. Both aligners can take a **GTF/GFF annotation file**, a “map” with the characteristics of the reference genome, in this case one with with exon boundaries and known splice junctions, but they can also identify splice junctions themselves. When providing an annotation file, it is critical that it matches the reference genome used for the mapping itself. 

## Bowtie2 and BWA

These aligners cannot detect intron junctions, and therefore they are used for samples derived from DNA, such as DNA-seq or ChIP-seq. They can, however, be used for RNA-seq too, as long as they are supplemented a **transcriptome**, where all introns have already been stripped. However, decoy-aware tools like Salmon are recommended over bowtie2 for RNA-seq, since they give less false positives.

A key difference between STAR/BWA and bowtie2 is that the first two use soft-clipping: ff the end of a read doesn't match the genome (maybe it's a bit of adapter or low quality), they just "ignore" those bases but keep them in the BAM file. Therefore, the mapping of a same sample with STAR/BWA and bowtie2 can give different scores, usually being higher in STAR/BWA. Due to this, bowtie2 is a better option for assays like ATAC-seq or CUT&RUN, especially in --end-to-end mode, where you want to be very strict about the detected sequences.

## Salmon/kallisto

Salmon and kallisto can work both as an aligner and a tool for quantification, giving information on how many reads come from each transcript or gene (**Transcripts per Million, TPM**). Instead of mapping to a whole genome, these tools “quasi-map” to a transcriptome index, meaning it has no introns. These can be obtained from Ensembl or GENCODE. It is good practice to combine this transcriptome with the whole genome as decoy to reduce erroneous mapping to unannotated regions.
Salmon not only is faster and lighter, it also corrects based on fragment GC content bias, transcript positional bias, and sequencing bias. However, it does not generate bam files, since it doesn´t map using the whole genome (this is why Salmon is sometimes referred to as a **pseudo-aligner**). It is therefore best to use it over the other aligners when the downstream analysis consists only of differential expression and clustering. For variant calling, splicing analysis, etc, an aligner that generates bam files is needed.

The main attributes given by these tools are NumReads and TPM. **NumReads** is the sum of reads assigned to a specific transcript, so it is not normalized and is used for differential expression analysis with tools like DESeq2, which have their own normalization algorithms. TPM, on the other hand, provides a normalized value that allows comparison between genes within a sample. It corrects for gene length (longer genes will have more reads) and for sequencing depth (so samples with unequal reads are comparable). TPM is for comparing Gene A vs Gene B within the same sample. Because the "Total Counts" denominator changes between samples, you should still use DESeq2 (which uses median-of-ratios normalization) for between-sample comparisons. TPM is not a substitute for proper statistical normalization.

## Selecting the Right Aligner

| Assay | Aligner | Reasoning |
|-------|---------|-----------|
| Standard RNA-seq (Eukaryotes) | STAR | It is splice-aware and handles introns via "N" operations in the CIGAR string |
| WGS/Variant Calling | BWA-MEM | It handles longer reads and indels more robustly |
| ATAC / CUT&RUN | Bowtie2 | Higher speed and specific end-to-end alignment |
| Prokaryotic RNA-seq | Bowtie2 | There are no introns in bacterial genomes |

