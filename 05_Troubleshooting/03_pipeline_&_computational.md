# Pipeline & Computational Troubleshooting

This section covers cases where downstream biological interpretation fails despite sequencing QC and mapping metrics appearing within acceptable ranges.

## RNA-seq

Even when sequencing QC and mapping metrics are within expected ranges, RNA-seq results can still appear inconsistent or biologically implausible during quantification and differential expression analysis. In these cases, the issue is typically not related to data quality, but to how reads are assigned, summarized, and statistically modeled downstream.

### Gene Quantification

After successful alignment, read counting tools such as featureCounts may produce unexpected or biologically inconsistent gene count distributions. A common cause is **incomplete or mismatched gene annotation**. If the GTF file does not match the genome build used for alignment, a substantial fraction of reads may map correctly but fail to be assigned to any feature. This results in inflated “unassigned” fractions and artificially reduced gene counts.

Additionally, gene quantification using featureCounts can produce unexpected or biased results depending on **parameter selection**. In most cases, issues are not caused by the data itself, but by mismatches between library properties and counting settings.

- **Strand-specific counting:** One of the most common sources of incorrect gene counts is a mismatch between library strandedness and the -s parameter used in featureCounts. If the strandedness setting does not match the library preparation protocol, reads may be incorrectly assigned or discarded.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| -s | 0 (unstranded)<br><br>1 (forward stranded)<br><br>2 (reverse stranded) | Strand specification does not match library protocol | Increased unassigned reads, loss of antisense signal, distorted gene expression profiles |

