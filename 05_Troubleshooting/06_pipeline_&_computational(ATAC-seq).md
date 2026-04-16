# Pipeline & Computational Troubleshooting (ATAC-seq)

Even when sequencing quality, alignment, and standard QC metrics are within expected ranges, ATAC-seq results can still appear inconsistent or biologically implausible during downstream analysis. In these situations, the issue is rarely the raw data itself, but how reads are translated into signal.

## TSS Enrichment Score

The TSS enrichment score is often the first indication that something is wrong despite otherwise acceptable QC metrics, as it directly reflects how well the signal has been processed and aligned to regulatory features.

A flat or weak TSS enrichment profile can be a consequence of the following:

- **Tn5 shift non applied:** As explained in the [ATAC-seq analysis](../04_Epigenomics/04_ATAC-seq_analysis.md), the transposases causes a 9 bp shif in the reads that needs to be considered during alignment. Failing to do so could lead to a misalignment of cut sites.
- **BAM filtering:** BAM files from different samples need to be properly filtered (especially by removing reads mapping to chrM) to prevent accumulation of background reads that might mask enrichment at TSS.

## Early Visualization in Genome Browsers

After BAM file processing and in parallel to TSS enrichment score inspection, it is good practice to study the signal in a genome browser such as IGV before proceeding to peak calling and downstream analysis.

Unlike CUT&RUN, ATAC-seq does not produce discrete protein binding sites but rather a continuous accessibility landscape. Therefore, interpretation relies on recognizing characteristic patterns of open chromatin rather than isolated peaks. A clean BAM file derived from a good ATAC-seq experiment should show:

- Sharp enrichment at promoters and known regulatory regions
- Clear depletion in heterochromatic regions
- Consistent signal patterns across replicates
- Well-defined peaks with low background signal

### Uniform signal across the whole genome

If signal appears broadly distributed with little contrast:

- **Presence of background reads:** BAM files not properly filtered are likely to show higher levels of background and therefore masked peaks.
- **Over-tagmentation:** Uncontrolled enzymatic activity can introduce widespread low-level accessibility, reducing contrast between open and closed chromatin.

### Lack of enrichment at expected regions

If promoters or known accessible loci show little to no signal:

- **Absence of Tn5 offset correction:** The signal may look blurry because reads are not properly shifted to the true cut positions.
- **Biologically real low accesibility:** The signal may truly be low in this condition, but it should be consistently low across all replicates if that is the case.

### Signal present but poorly defined

If enrichment exists but lacks sharp peaks:

- **Missing Tn5 shifting** results in blurred accessibility profiles.
- **Residual background reads** reduce signal-to-noise ratio.

### Enrichment in unexpected regions

If strong signal appears outside known regulatory elements:

- **Genome build:** The genome build used for visualization must match the one used in the mapping step, or the peaks will seem to appear in unexpected regions.
- **Incorrect filtering:** if not removed. Reads may accumulate in repetitive or blacklist regions if not removed. These regions can produce convincing but non-reproducible peaks.




