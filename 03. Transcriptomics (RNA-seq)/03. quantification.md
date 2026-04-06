# Quantification of reads

The bam files generated during the alignment contain the coordinates of the detected genes, but not the number of copies that were detected. To convert the reads into a digital **count matrix**, the go-to option in the feature of the Subread binary package, [featureCounts](https://subread.sourceforge.net/featureCounts.html). Here is the [link](https://bioconductor.org/packages/release/bioc/html/Rsubread.html) for R users, where featureCounts is wrapped in the Bioconductor package Rsubread. It is very fast and memory efficient, and it uses a GTF annotation file to define the boundaries between exons and genes. The 3 critical settings that must be taken into account when using featureCounts are the following:

- **Strandedness** (unstranded, forward stranded, or reverse stranded (standard dUTP/Illumina workflow): Picking the wrong one will lead to a massive decrease in the count number.
- **Multi-mapping:** Usually, ignore reads that map to multiple places are ignored to avoid "noise," but in some cases (like repetitive elements), it might be needed to switch this on.
- **Paired-end:** Must be switched on when using paired-end reads.

The manual for the featureCounts package can be found [here](https://subread.sourceforge.net/SubreadUsersGuide.pdf).

This package outputs a tab-delimited file with the number of reads, where rows are Gene IDs and columns are your individual samples (**count matrix**).
