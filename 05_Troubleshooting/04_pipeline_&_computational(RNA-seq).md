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

- This is often due to an **incorrect selection in the strandedness (`-s`) parameter**, e.g., the strand specification (`s-0` for unstranded, `s-1` for forward stranded, and `s-2` for reverse stranded) does not match the library generation.
- This can also be caused by an **incorrect feature type selection (`-t`)** or a mismatch with the one present on the GTF file. The standard in RNA-seq analysis is `-t exon`: if the GTF contains exon information under a different name, a mismatch will happen and reads will not be assigned correctly.
- Finally a **mismatch with attribute type (`-g`)**. The -t flag works in tandem with the -g flag; if `-t exon` is used, the `-g` flag must point to an attribute that exists within the exon lines (e.g., gene_id).

### Gene counts lower than expected across all samples

If overall expression levels appear systematically reduced, this often indicates a structural issue in counting.

- **Choosing the wrong feature type (`-t`)** leads to underestimation of gene expression. While, as mentioned above, `-t exon` is the standard in most RNA-seq pipelines, different experimental aims require a specific selection. For example, if the goal is to detect unspliced transcripts, `-t gene` should be used instead of `-t exon`.
- **Incorrect attribute type (`-g`) selection** can fragment counts across multiple identifiers, leading to gene count underestimation.

### Inflated library sizes or unusually high counts 

If total counts per sample are unexpectedly high, especially in paired-end data, counting may be incorrect.

- When analysing data from paired-end sequencing, **paired mode (`-p`)** should be enabled, or paired reads will be treated as independent single reads, leading to double counting.
- The parameter **`--countReadPairs`** also needs to be enabled, to ensure that fragments from paired-end reads are counted as single units.

### Loss of reads in specific regions

If specific regions or the overall dataset show reduced counts, filtering parameters such as the following may be too strict:

- Selecting the **wrong fragment length thresholds (`-d`, `-D`)** can lead to the loss of valid fragments, but also to the inflation of counts if the lower threshold is too small.
- Enabling **proper pairing requirement (`-B`)** can lead to the loss of valid partially mapped fragments
- While **chimeric read exclusion (`-C`)** is usually recommended, there are very specific contexts in which doing so might remove potentially biologically relevant reads.

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

### High proportion of unmapped or low-confidence reads

If Salmon reports a low overall mapping rate (typically visible in the salmon_quant.log file):

- If a **decoy-aware index was not built** (using the genome sequence as decoy), reads originating from unannotated genomic regions may map spuriously to the nearest transcript, inflating some counts while reducing overall confidence. A decoy-aware index using `salmon index` with the `--decoys` flag should always be built (or downloaded if computational power is limited)
- The **transcriptome reference may not match the genome build used upstream**. Salmon requires a transcriptome FASTA, not a genome FASTA — using the wrong file type will produce near-zero mapping.
- **Strandedness** must be specified correctly via `--libType`. Running a stranded library without specifying it causes Salmon to discard a large fraction of reads as incompatible. It is good practice to use `--libType A` for automatic detection on a small pilot subset first and then lock it in for the full run.

### Inflated or unexpectedly high expression estimates

If overall expression appears inflated or disproportionately concentrated in a subset of transcripts, this may indicate mapping ambiguity or missing reference information.

- **Incomplete transcriptomes** can cause reads to map spuriously to similar available transcripts.
- Typically caused by **multi-mapping reads being distributed across similar isoforms**. Salmon's algorithm will assign fractional counts to all compatible transcripts, so highly similar isoforms will each receive a share of ambiguous reads. Therefore isoform-level estimates are less reliable than gene-level ones. To collapse these transcripts to gene level, tools like [tximeta](https://bioconductor.org/packages/release/bioc/html/tximeta.html) or [tximport](https://bioconductor.org/packages/release/bioc/html/tximport.html) are often employed before differential expression analysis.

### Gene-level counts discordant with featureCounts results

Salmon and featureCounts operate on fundamentally different inputs — a transcriptome vs a genome — and use different assignment logic. Some discordance is expected, particularly for multi-isoform genes. However, if such discordance is extreme (order-of-magnitude differences for most genes), the following should be checked:

- **Correct strandedness** was specified in both tools
- **Same annotation version** was used to build the Salmon index and the featureCounts GTF.

### TPM values look reasonable but DESeq2 results are poor

Salmon's **TPM output should never be fed directly into DESeq2**. DESeq2 requires raw, unnormalized counts. tximport or tximeta are typically used to import Salmon's `quant.sf` files, which correctly handles the conversion to estimated counts while preserving the information DESeq2 needs for its own normalization. Passing TPM values into DESeq2 will produce results that appear to run without error but are statistically invalid.

### Quick diagnostic guide

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Low overall mapping rate | Wrong reference type or missing decoy | Transcriptome vs genome FASTA, decoy-aware index |
| Large fraction of incompatible reads | Strandedness mismatch | `--libType`, library prep protocol |
| Inflated counts in lowly expressed transcripts | Multi-mapping across similar isoforms | Isoform similarity, collapse to gene level |
| Discordance with featureCounts | Expected; different assignment logic | Strandedness consistency, annotation version |
| DESeq2 results look wrong after Salmon | TPM used instead of counts | Use tximport/tximeta to import `quant.sf` |

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
