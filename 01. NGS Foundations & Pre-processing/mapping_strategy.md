## Mapping the Reads to a Reference Genome

Once the sequencing data has been cleaned and validated through QC, the next step in the primary analysis pipeline is **mapping** or **alignment**. This is the process of determining the original coordinate of each DNA fragment by comparing the sequences in the FASTQ files to a known **reference genome**. Choosing the correct reference and the appropriate alignment algorithm is critical, as any errors at this stage—such as mismatches or multi-mapping reads—will propagate through to the final analysis.

# Reference Genomes

The most widely used versions of human reference genome are GRCh37 and GRCh38, with the latter being more recent and constantly updated. However, if the data or database used in the experiment is still on version GRCh37, then the same reference genome should be used. It is important to note that these are the **Ensembl** reference genomes. Their **USCS** counterparts (hg19 for GRCH37 and hg38 for GRCh38) are equally valid but differ in naming and structure. While in the Ensembl genomes chromosomes are named with only numbers, USCS uses chr1, chr2, etc. Some alignment tools require exact chromosome names, or they will fail or mismatch, so it is essential to choose one format and stick with it for the whole analysis.

•	Ensembl is a genome browser and annotation platform maintained by EMBL–EBI. It provides genome assemblies, gene annotations, variation data, and other reference resources for many species.
•	GENCODE is a specific annotation project that focuses on producing high-quality, manually curated gene models — initially for human and mouse genomes.

GENCODE and Ensembl are now technically the same for human and mouse. Since 2018, Ensembl uses the GENCODE gene set as its default, but the latter includes a lot of "long non-coding RNAs" (lncRNAs) and pseudogenes that standard RefSeq might ignore. This is why RNA-seq scientists almost always prefer GENCODE/Ensembl over UCSC/RefSeq.

| Version	Source	| Common Alias	| Chromosome Naming	| Annotation | Source	Notes |
|-----------------|---------------|-------------------|------------|--------------|
| GRCh37 |	Ensembl	| —	1, 2, 3…MT |	Ensembl 75 / GENCODE v19	| Older but stable |
| hg19 |	UCSC	| —	chr1, chr2…chrM	| RefSeq / GENCODE v19	| Equivalent to GRCh37 |
| GRCh38 |	Ensembl	| —	1, 2, 3…MT	| Ensembl ≥76 / GENCODE v40	| Current standard |
| hg38 |	UCSC	| —	chr1, chr2…chrM	| RefSeq / GENCODE v40	| Equivalent to GRCh38 |
