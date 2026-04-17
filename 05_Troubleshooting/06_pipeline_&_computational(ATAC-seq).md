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

**Note:** Even though BAM file inspection would theoretically be performed before coverage files generation, many pipelines skip this step due to the heavy computational power required to open and zoom in into BAM files, which are usually in the Gb range. However, if errors are spotted in downstream steps (coverage track generation, peak calling, etc), visualizing BAM files in a genome browser might clarify if the problem occured during the alignment process or in subsequent steps.

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

## Coverage File Generation

After BAM processing, normalized coverage tracks (usually BigWig files generated with deepTools bamCoverage) should be inspected before peak calling. These tracks provide a direct view of chromatin accessibility and often reveal issues that are not apparent from alignment or QC metrics alone.

A correct coverage track should show clear enrichment at promoters and regulatory regions, low background signal, and consistent profiles across replicates.

Peak Calling (MACS3 in ATAC-seq)
After alignment and preprocessing, MACS3 peak calling may still produce biologically inconsistent or unstable peak sets even when QC metrics look acceptable. In ATAC-seq, this is often due to mismatch between fragment representation, signal structure, and MACS3 assumptions, rather than the peak caller itself.
A reliable peak set should show strong overlap with promoter regions, sharp peak boundaries, and consistency across replicates. When this is not observed, the first step is to assume a modeling or input inconsistency, not a biological absence of signal.
Fragment structure mismatch (paired-end vs single-end assumptions)
A common source of distorted peak profiles is incorrect handling of fragment structure.
If paired-end data is not treated in BAMPE mode, fragments are artificially reconstructed from reads rather than actual fragment lengths.
This leads to broader, less resolved peaks and reduced peak height at true accessible sites.
Conversely, applying single-end shift/extsize logic to paired-end data introduces artificial centering and can distort true accessibility boundaries.
If fragment handling is incorrect:
peaks become broader than expected
promoter enrichment becomes weaker relative to background
replicate agreement decreases even if mapping quality is high
Tn5 positioning inconsistency (single-end workflows)
If single-end ATAC-seq data is processed without proper Tn5 shift correction, MACS3 will not be operating on true cut-site positions.
This results in positional smearing of accessibility signal
TSS enrichment may appear reduced even when raw signal is strong
Peaks may appear offset relative to promoter annotations in genome browsers
This is not a MACS3 limitation, but a misalignment between biological cut sites and computational representation
Excessively weak or sparse peak sets
If MACS3 produces very few peaks despite strong alignment and reasonable depth:
overly strict q-value thresholds may be filtering true signal
prior filtering (duplicates, mitochondrial reads) may have removed too much usable signal
fragment signal may already be degraded (low TSS enrichment, low complexity library)
This typically indicates a loss of signal before peak calling, not a failure of MACS3 itself.
Expected consequence:
promoter peaks disappear first
distal regulatory elements fail to be detected
replicate overlap becomes unstable
Inflated or noisy peak sets
If peak calling produces an unusually large number of weak or scattered peaks:
residual mitochondrial or unmapped background reads may still be present
PCR duplicates may be inflating low-confidence regions
blacklist regions may not have been removed
over-tagmentation can create widespread low-level accessibility
In these cases, MACS3 will still generate peaks, but they will reflect technical background structure rather than chromatin accessibility
Typical symptoms:
peaks distributed across non-regulatory regions
weak enrichment in promoters compared to genome-wide background
poor reproducibility across replicates
Loss of peak definition (broad or blurred peaks)
If peaks are present but lack sharp boundaries:
incorrect fragment modeling reduces resolution of accessibility signal
absence of summit calling reduces positional precision
high background signal reduces signal-to-noise ratio
This leads to:
diffuse promoter peaks instead of sharp enrichment
difficulty separating closely spaced regulatory elements
reduced interpretability in downstream annotation
Genome mismatch or annotation inconsistency
If peaks appear in unexpected genomic regions or do not align with known regulatory features:
genome build mismatch between alignment and peak calling/visualization will shift peak coordinates
inconsistent reference annotation may lead to incorrect interpretation of peak location
coordinate sorting or file conversion issues can misrepresent peak positions
This typically manifests as:
apparent “novel” peaks that are not reproducible
promoter signals appearing in intronic or intergenic misaligned regions
MACS3 interpretation principle
In ATAC-seq, MACS3 does not infer biology—it formalizes accessibility structure already present in the BAM file.
Therefore:
weak TSS enrichment → weak or absent peaks regardless of parameters
high background → noisy peak set regardless of thresholding
correct fragment representation → stable and reproducible peaks
Pre–peak-calling sanity check (critical)
Before trusting MACS3 output, the following should already be consistent:
strong and narrow TSS enrichment
expected fragment size distribution (nucleosomal patterning)
low mitochondrial contamination
controlled duplicate levels
clear promoter/enhancer structure in genome browser
If these are not met, MACS3 tuning will not fix downstream structure.


