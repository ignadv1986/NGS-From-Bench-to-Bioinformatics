# Pipeline & Computational Troubleshooting (ATAC-seq)

Even when sequencing quality, alignment, and standard QC metrics are within expected ranges, ATAC-seq results can still appear inconsistent or biologically implausible during downstream analysis. In these situations, the issue is rarely the raw data itself, but how reads are translated into signal.

## TSS Enrichment Score

The TSS enrichment score is often the first indication that something is wrong despite otherwise acceptable QC metrics, as it directly reflects how well the signal has been processed and aligned to regulatory features.

A flat or weak TSS enrichment profile can be a consequence of the following:

- **BAM filtering:** BAM files from different samples need to be properly filtered (especially by removing reads mapping to chrM) to prevent accumulation of background reads that might mask enrichment at TSS.
- **Over-Digitization:** If the Tn5 concentration is too high for the number of nuclei, it will force its way into the edges of nucleosomes, leading to a broad plateau instead of a sharp spike in the TSS enrichment plot.
- **Low Non-Redundant Fraction (NFR):** If the TSS score is low and the duplication rate is high (low NRF), that is a signal of an under-sampled library. This often happens if the transposition was successful but the DNA recovery (PCR/clean-up) was poor, leading to a small amount of reads being repetaedly sequenced. Because the TSS score relies on a ratio of signal-to-background, a lack of unique diverse reads can actually make the TSS signal look artificially weak or statistically insignificant.

### Quick diagnostics guide

| Symptom | Likely Cause | What to Check|
| :--- | :--- | :--- |
| Flat or low TSS enrichment profile | BAM filtering issues | Check for presence of chrM reads, PCR duplicates, or suboptimal filtering strategy in the final BAM |
| Broad plateau instead of sharp spike | Over-transposition (Over-digitization) | Check Tn5-to-nuclei ratio. Excessive enzyme forces cuts into nucleosome edges, blurring the TSS signal |
| Weak enrichment + High duplication | Low Library Complexity (NRF) | Check the Non-Redundant Fraction. Signal is likely masked by under-sampling due to poor DNA recovery post-transposition |
| High variability between samples | Inconsistent Preprocessing | Verify that uniform filtering (e.g., same MAPQ thresholds and duplicate removal) was applied across the entire cohort |

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
- **Incorrect filtering:** If not removed, reads may accumulate in repetitive or blacklist regions. These regions can produce convincing but non-reproducible peaks.

### Quick diagnostics guide

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Uniform signal across genome (low contrast) | Background contamination or over-tagmentation | BAM filtering, chrM removal, fragment distribution |
| Lack of enrichment at promoters or known loci | Missing Tn5 shift or biological absence of accessibility | Alignment shift, replicate consistency |
| Signal present but poorly defined | Suboptimal processing or mixed fragment signal | Fragment size distribution, BAM quality |
| High signal in heterochromatic regions | Poor filtering or alignment artifacts | MAPQ filtering, blacklist regions |
| Inconsistent patterns between replicates | Technical variability | Replicate concordance, library quality |

## Coverage File Generation

After BAM processing, normalized coverage tracks (usually BigWig files generated with deepTools bamCoverage) should be inspected before peak calling. These tracks provide a direct view of chromatin accessibility and often reveal issues that are not apparent from alignment or QC metrics alone.

### Uniform signal across the whole genome

If coverage appears broadly distributed with little contrast between regions:

- **Incomplete filtering of reads:** Incomplete BAM filtering increases baseline signal and masks true accessibility.
- **Over-tagmentation:** Excessive enzymatic activity can generate widespread low-level accessibility signal.
- **Scaling issues:** Lack of proper normalization with CPM or RPKM may make high-depth samples appear uniformly elevated. This behavior is controlled by the `--normalizeUsing` parameter in bamCoverage, and inconsistent settings across samples can introduce artificial differences in signal intensity.

### Signal intensity differs strongly between samples

If one sample appears globally stronger or weaker than others:

- **Missing or inconsistent normalization:** All samples should be normalized using the same method, or library sizes and other factors can affect the scaling and lead to result misinterpretation.
- **chrM contamination:** High mitochondrial content can compress nuclear signal in affected samples, so it should be equally removed in all samples.

### Lack of enrichment at promoters or known regulatory regions

If expected accessible regions show weak or no signal:

- **Low accesibility:** Biologically true accesibility can occur due to specific treatments or vary between cell lines, but it should be consistent across replicates.
- **Missing Tn5 correction:** Signal becomes blurred and less concentrated at true cut sites, sometimes making it impossible to detect enrichment at such sites.

### Signal present but poorly defined (blurred or wide peaks)

If enrichment exists but lacks sharp boundaries:

- **Incorrect bin size:** Large bins oversmooth signal; very small bins introduce noise. The default for deepTools bamCoverage is 50 bp, but sometimes this can prevent detection of narrow peak information. Bin size can be modified in bamCoverage with `--binSize` and 1-10 bp ranges can be used to detect these peaks 
- **Fragment handling issues:** Treating paired-end data as single-end reduces resolution.
- **Missing Tn5 correction:** Signal may appear slightly blurred around regulatory regions, but this effect is often subtle at the coverage level and easier to detect in TSS enrichment profiles or peak definition.

### Enrichment in unexpected regions

If strong signal appears outside known regulatory elements:

- **Genome build mismatch:** Coverage does not align with annotation or browser reference
- **Incomplete filtering:** Reads accumulating in blacklist or repetitive regions
- **Multimapping reads retained:** Artificial signal in low-complexity regions

### Quick diagnostics guide

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Uniform signal across genome (low contrast) | Incomplete filtering or over-tagmentation | BAM filtering (chrM, duplicates), fragment size distribution |
| Uniformly elevated signal in one sample | Missing or incorrect normalization | CPM/RPKM normalization, library size |
| Strong differences in signal intensity between samples | Inconsistent normalization or chrM contamination | Normalization method consistency, mitochondrial read fraction |
| Weak or no enrichment at promoters / regulatory regions | Low biological accessibility or missing Tn5 correction | Replicate consistency, Tn5 shift application |
| Signal present but blurred or poorly defined | Missing Tn5 correction or large bin size | Read shifting, `--binSize` parameter |
| Peaks overly broad | Oversmoothing or improper fragment handling | Bin size, paired-end vs single-end treatment |
| Noisy or fragmented signal | Very small bin size or low coverage | `--binSize`, sequencing depth |
| Enrichment in unexpected regions | Genome mismatch or poor filtering | Genome build, blacklist removal |
| Strong signal in repetitive regions | Multimapping reads retained | Alignment settings, MAPQ filtering |
| Coverage does not match BAM visualization | Incorrect bamCoverage parameters | Consistency between BAM processing and coverage generation |

## Peak Calling

After alignment and preprocessing, MACS3 peak calling may still produce biologically inconsistent or unstable peak sets even when QC metrics look acceptable. In ATAC-seq, this is often due to mismatch between fragment representation, signal structure, and MACS3 assumptions, rather than the peak caller itself.

When inspecting the narrowPeak files generated in the peak calling step, a reliable peak set should show strong overlap with promoter regions, sharp peak boundaries, and consistency across replicates. Additionally, it should ressemble what can be seen in the coverage files.

### Broad or poorly defined peaks

- **Missing Tn5 shift correction:** Without the standard +4/-5bp adjustment, the signal from the forward and reverse strands remains physically separated by the 9 bp duplication created during transposition. This causes the peak caller to see two slightly offset sub-populations of reads rather than a single, unified insertion site. Consequently, MACS3 may "smear" the signal, resulting in artificially increased peak widths and a loss of the sharp, "pointy" summits characteristic of high-quality ATAC-seq data. This lack of precision can obscure fine-scale genomic features, such as transcription factor footprints, and dilute the statistical significance of narrow regulatory elements.


