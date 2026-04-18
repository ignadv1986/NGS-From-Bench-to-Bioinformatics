# Pipeline & Computational Troubleshooting (RNA-seq)

Even when sequencing QC and mapping metrics are within expected ranges, RNA-seq results can still appear inconsistent or biologically implausible during quantification and differential expression analysis. In these cases, the issue is typically not related to data quality, but to how reads are assigned, summarized, and statistically modeled downstream.

## Gene Quantification (featureCounts)

After successful alignment, read counting tools such as featureCounts may produce unexpected or biologically inconsistent gene count distributions. A common cause is **incomplete or mismatched gene annotation**. If the GTF file does not match the genome build used for alignment, a substantial fraction of reads may map correctly but fail to be assigned to any feature. This results in inflated “unassigned” fractions and artificially reduced gene counts, and should therefore be the first thing to be checked.

<br>

<div align="center">
  
  <img src="../Figures/GTF.png" width="800">
  <br>
  <em>"GTF file example", by Cjpenaponton.  Licensed under Creative Commons Attribution-Share Alike 4.0 International (https://creativecommons.org/licenses/by-sa/4.0/deed.en).</em> 
</div>

<br>

In particular, the presence and consisten annotation of selected feature type (e.g., exon) and attribute (e.g., gene_id) in the GTF file should be verified.

Even when the annotation is correct, unexpected results can arise when **counting assumptions** do not match the properties of the sequencing library.

### High proportion of unassigned reads

If the correct GTF file was used, but the proportion of unassigned reads is still too high (>25-30%):

- This is often due to an incorrect selection in the strandedness (`-s`) parameter, e.g., the strand specification (`s-0` for unstranded, `s-1` for forward stranded, and `s-2` for reverse stranded) does not match the library generation.
- This can also be caused by an incorrect feature type selection (`-t`) or a mismatch with the one present on the GTF file. The standard in RNA-seq analysis is `-t exon`: if the GTF contains exon information under a different name, a mismatch will happen and reads will not be assigned correctly.
- Finally a mismatch with attribute type (`-g`). The -t flag works in tandem with the -g flag; if `-t exon` is used, the `-g` flag must point to an attribute that exists within the exon lines (e.g., gene_id).

### Gene counts lower than expected across all samples

If overall expression levels appear systematically reduced, this often indicates a structural issue in counting.

- Choosing the wrong feature type (`-t`) leads to underestimation of gene expression. While, as mentioned above, `-t exon` is the standard in most RNA-seq pipelines, different experimental aims require a specific selection. For example, if the goal is to detect unspliced transcripts, `-t gene` should be used instead of `-t exon`.
- Incorrect attribute type (`-g`) selection can fragment counts across multiple identifiers, leading to gene count underestimation.

### Inflated library sizes or unusually high counts 

If total counts per sample are unexpectedly high, especially in paired-end data, counting may be incorrect.

- When analysing data from paired-end sequencing, paired mode (`-p`) should be enabled, or paired reads will be treated as independent single reads, leading to double counting.
- The parameter `--countReadPairs` also needs to be enabled, to ensure that fragments from paired-end reads are counted as single units.

### Loss of reads in specific regions

If specific regions or the overall dataset show reduced counts, filtering parameters such as the following may be too strict:

- Selecting the wrong fragment length thresholds (`-d`, `-D`) can lead to the loss of valid fragments, but also to the inflation of counts if the lower threshold is too small.
- Enabling proper pairing requirement (`-B`) can lead to the loss of valid partially mapped fragments
- While chimeric read exclusion (`-C`) is usually recommended, there are very specific contexts in which doing so might remove potentially biologically relevant reads.

### Quick diagnostic guide

<br>

<div align="center">

| Symptom | Likely cause | Parameter(s) to check |
| :--- | :--- | :--- |
| High proportion of unassigned reads despite good mapping | Incorrect strand specification | `-s` |
| Gene counts lower than expected across all samples | Mismatch between annotation and feature definition | `-t`, `-g` |
| Unexpectedly high library sizes / inflated counts | Paired-end reads counted incorrectly as single reads | `-p`, `--countReadPairs` |
| Loss of reads in specific regions or globally reduced counts | Overly strict fragment filtering | `-d`, `-D`, `-B`, `-C` |
</div>

<br>

## Gene Quantification (Salmon)

After quantification, Salmon can produce expression estimates that appear inconsistent or biologically implausible even when mapping rates are high. These issues are typically not due to sequencing quality, but rather to **mismatches between the transcriptome reference, library properties, and the assumptions used during quantification**.

Because Salmon operates at the transcript level using probabilistic assignment, small inconsistencies in reference or library structure can propagate into large differences at the gene level.

### High proportion of low-confidence or unstable transcripts

If a large fraction of transcripts show unstable or unexpectedly low abundance estimates, this often indicates ambiguity in read assignment.

- This can occur when the transcriptome contains many highly similar isoforms, making it difficult to uniquely assign reads.
- It can also indicate an incomplete or overly simplified transcriptome reference.

### Unexpected expression in transcripts that should not be active

If transcripts are detected in conditions where they are not biologically expected, this may indicate issues with reference or mapping specificity.

- Incomplete or outdated transcriptome annotations can lead to reads being assigned to incorrect transcript models.
- High sequence similarity between transcripts can cause probabilistic assignment to favor incorrect isoforms.

### Systematic strand-related or directional bias

If expression appears biased toward the wrong strand or inconsistent with library preparation expectations, this suggests a mismatch in library type assumptions.

- Reads may be assigned in the wrong orientation if library structure is incorrectly specified or inferred.
- This can distort both transcript-level estimates and downstream gene summarization.

### Inflated or unexpectedly high expression estimates

If overall expression appears inflated or disproportionately concentrated in a subset of transcripts, this may indicate mapping ambiguity or missing reference information.

- Incomplete transcriptomes can cause reads to map spuriously to similar available transcripts.
- Lack of decoy-aware indexing can increase incorrect assignments from unannotated regions.

### Quick diagnostic guide

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| High fraction of unstable or low-confidence transcripts | Transcriptome ambiguity or incompleteness | Transcriptome reference quality |
| Expression detected in unexpected transcripts | Incomplete or outdated annotation | Transcriptome version consistency |
| Strand inconsistencies in expression patterns | Library structure mismatch | Library preparation assumptions |
| Inflated expression in subsets of transcripts | Mapping ambiguity or missing decoy sequences | Reference completeness, decoy-aware index |

## Differential Expression (DESeq2)

Poor experimental design structure, technical variation, or incorrect model specification can all lead to result misinterpretation during differential exoression analysis. Therefore, they need to be properly taken into account when running DESeq2 after read counting.

### Unexpected separation of samples

If samples cluster primarily by non-biological variables rather than the intended condition, the differential expression results are likely being driven by technical variation.

- During the experimental design stage, it is strongly recommended to include at least one replicate of each condition in each batch. If each condition is present in only one batch, DESeq2 cannot distinguish technical from biological effects, making differential expression results unreliable.
- Relevant sample information, such as condition, replicates, and batch, should be included in the metadata table provided alongside the count matrix. Otherwise, DESeq2 may attribute systematic differences between samples to the condition of interest, even when those differences are technical.

### Very few significant genes or unstable results across comparisons

If the analysis yields very few significant genes, or results change substantially with small adjustments, the model may be underpowered or unstable.

- To increase analysis power, the number of replicates per conditions should be appropiate. In addition, replicates should show consistent expression patterns at the count level (e.g., clustering together in PCA or correlation analyses), as high variability within conditions can obscure true differences between groups.
- The presence of outlier samples can inflate variance estimates and reduce the ability to detect differential expression, and individual samples with extreme values can disproportionately affect results.

### Global shifts in expression across conditions

If one condition shows a systematic increase or decrease in expression across many genes, normalization may be affected.

- While some biological conditions can influence many genes, DESeq2 assumes that the majority of genes are not differentially expressed. When this assumption is violated, normalization can become unreliable.
- This can occur when a small number of highly expressed genes dominate the library in one condition, shifting the relative abundance of all other genes (composition bias).
- Although differences in sequencing depth are typically identified during earlier QC steps, extreme imbalances combined with skewed expression profiles can still distort relative expression estimates after normalization.

### Quick diagnostic guide

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Samples cluster by technical factors instead of condition | Technical variation driving signal; missing or confounded batch information | Metadata table (condition, batch), sample distribution across batches |
| Very few significant genes or unstable results | Low statistical power or high variability between replicates | Number of replicates, sample similarity (PCA, correlation) |
| Inconsistent or biologically implausible fold changes | Incorrect comparison or incomplete sample annotation | Reference level, contrasts, metadata completeness |
| Global shifts in expression across conditions | Violation of normalization assumptions due to composition bias | Presence of highly expressed genes, overall expression distribution |

## Peak Quantification

After peak calling, ATAC-seq analysis typically proceeds by quantifying reads overlapping a set of genomic regions (peaks) using featureCounts. Unlike in RNA-seq, counts in ATAC-seq are derived from genomic intervals rather than annotated genes, and results depend strongly on how the peak set is defined and applied across samples.

Unexpected or inconsistent count distributions at this stage are most often due to peak definition and counting configuration, rather than alignment issues.

### Inconsistent counts across samples despite similar signal
If samples with similar coverage profiles produce very different count values:
Inconsistent peak set usage:
All samples must be quantified against the same set of genomic regions. Using sample-specific peaks leads to non-comparable counts.
Improper peak merging strategy:
Overly broad or poorly defined consensus peaks can dilute signal, while overly narrow peaks may miss relevant fragments.
Systematically low counts across all peaks
If counts appear lower than expected:
Fragment counting not enabled in paired-end data:
If paired-end reads are not handled correctly, fragments may be undercounted or inconsistently represented.
Stringent overlap requirements:
Requiring full overlap or strict assignment can exclude valid fragments that partially overlap peak regions.
Inflated counts or unusually high signal in peaks
If counts appear unexpectedly high:
Paired-end reads counted as independent reads:
If fragment-based counting is not enforced, each mate may be counted separately.
Overly broad peak regions:
Large peaks accumulate more reads, inflating counts and reducing resolution between regions.
Loss of signal in specific peaks
If expected accessible regions show weak or missing counts:
Poorly defined consensus peaks:
Peaks that do not accurately capture the true accessible region may miss signal during counting.
Inconsistent preprocessing:
Differences in duplicate removal or filtering across samples can affect read assignment.
Quick diagnostics guide
| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Counts differ strongly between similar samples | Inconsistent peak sets | Consensus peak definition |
| Low counts across peaks | Fragment counting or strict overlap rules | Paired-end handling, assignment settings |
| Inflated counts in peaks | Double counting or broad peaks | Paired-end mode, peak width |
| Missing signal in expected regions | Poor peak definition | Peak set quality, preprocessing consistency |
Differential Accessibility (DESeq2)
Once peak counts are generated, DESeq2 is used to identify regions with differential accessibility between conditions. At this stage, issues are rarely caused by sequencing or alignment, but rather by experimental design, normalization assumptions, and properties of the peak set.
Unexpected separation of samples
If samples cluster by technical variables rather than biological condition:
Batch effects not modeled:
Batch or technical variables must be included in the design formula when present.
Confounded experimental design:
If conditions are not distributed across batches, technical and biological effects cannot be separated.
Very few significant regions or unstable results
If few regions are detected or results change easily:
Low replicate number or high variability:
ATAC-seq is particularly sensitive to variability in accessibility across replicates.
Inconsistent peak set:
Poorly defined or noisy peaks reduce statistical power.
Global shifts in accessibility
If one condition appears globally more or less accessible:
Violation of normalization assumptions:
DESeq2 assumes most regions are not differentially accessible. Large-scale chromatin changes can break this assumption.
Composition bias:
A subset of highly accessible regions dominating one condition can skew normalization.
Strong differences driven by a subset of peaks
If results are dominated by a small number of regions:
Outlier peaks:
Extremely high-count regions can disproportionately influence normalization and dispersion estimates.
Technical artifacts in peak set:
Peaks overlapping problematic regions (e.g., repeats, blacklist regions) may bias results.
Quick diagnostics guide
| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Samples cluster by batch instead of condition | Technical variation not modeled | Design formula, metadata |
| Few significant regions | Low power or noisy peak set | Replicates, peak quality |
| Global accessibility shifts | Normalization assumption violation | Count distribution, dominant peaks |
| Results driven by few regions | Outlier peaks or artifacts | Peak inspection, filtering 
