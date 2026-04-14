# Pipeline & Computational Troubleshooting

This section covers cases where downstream biological interpretation fails despite sequencing QC and mapping metrics appearing within acceptable ranges.

## RNA-seq

Even when sequencing QC and mapping metrics are within expected ranges, RNA-seq results can still appear inconsistent or biologically implausible during quantification and differential expression analysis. In these cases, the issue is typically not related to data quality, but to how reads are assigned, summarized, and statistically modeled downstream.

### Gene Quantification

After successful alignment, read counting tools such as featureCounts may produce unexpected or biologically inconsistent gene count distributions. A common cause is **incomplete or mismatched gene annotation**. If the GTF file does not match the genome build used for alignment, a substantial fraction of reads may map correctly but fail to be assigned to any feature. This results in inflated “unassigned” fractions and artificially reduced gene counts.

Additionally, gene quantification using featureCounts can produce unexpected or biased results depending on **parameter selection**. In most cases, issues are not caused by the data itself, but by mismatches between library properties and counting settings.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| Strand-specific Counting | -s 0 (unstranded)<br><br>-s 1 (forward stranded)<br><br> -s 2 (reverse stranded) | Choice of a strandness method not matching the library preparation protocol | -Increased proportion of unassigned reads<br><br>-Apparent loss of signal in antisense or overlapping genes
-Reduced correlation with expected gene expression profiles |
