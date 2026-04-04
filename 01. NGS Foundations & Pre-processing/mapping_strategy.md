# Mapping the Reads to a Reference Genome

Once the sequencing data has been cleaned and validated through QC, the next step in the primary analysis pipeline is **mapping** or **alignment**. This is the process of determining the original coordinate of each DNA fragment by comparing the sequences in the FASTQ files to a known **reference genome**. Choosing the correct reference and the appropriate alignment algorithm is critical, as any errors at this stage—such as mismatches or multi-mapping reads—will propagate through to the final analysis.

## Reference Genomes

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

## Genome Indexing

Without an index, the aligner would have to read the entire 3.2 billion base pairs of the human genome for every single read in the FASTQ files. With 400 million reads, the analysis would take years. **Indexing** reduces this to minutes or hours. This involved turning the reference FASTA file into a fast-searchable database (index) of short, overlapping subsequences, known as **k-meres** (sequences of length *k*). 
Importantly, each aligner generates a different indexing, and they are not compatible. However, generating an index is a very computationally heavy task, so it is good practice to download an indexed genome (Illumina iGenomes, AWS, iGenomes, or Ensembl/USCS FTP) that can be used with the chosen aligner. 

## The Mapping Process

Aligners take short sequences (**seeds**) from the sequencing reads and try to find candidate alignment locations in the index. Once the aligner has "anchored" the read to a specific spot, it attempts to align the remainder of the read to the adjacent genomic coordinates, allowing for a predefined number of mismatches or gaps (indels). This is called the **seed-and-extend method**, and it is essential because, if the aligner tried to find an exact 150-base match in the 3-billion-base human genome, it would almost always fail, due to the presence of point mutations and sequencing errors. The aligner calculates an **alignment score** by rewarding matches and applying "penalties" for mismatches and indels. If a sequences maps to several locations, the aligner chooses the one with the highest alignment score. If multiple locations have the same top score, the read is considered **multi-mapped**, which significantly lowers its Mapping Quality (MAPQ) score (see below).

