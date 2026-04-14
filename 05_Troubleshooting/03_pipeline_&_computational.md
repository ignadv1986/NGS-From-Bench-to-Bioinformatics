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

- This is often due to an incorrect selection in the strandedness (*-s*) parameter, e.g., the strand specification (*s-0* for unstranded, *s-1* for forward stranded, and *s-2* for reverse stranded) does not match the library generation.
- This can also be caused by an incorrect feature type selection (*-t*) or a mismatch with the one present on the GTF file. The standard in RNA-seq analysis is *-t exon*: if the GTF contains exon information under a different name, a mismatch will happen and reads will not be assigned correctly.
- Finally a mismatch with attribute type (*-g*). The -t flag works in tandem with the -g flag; if *-t exon* is used, the *-g* flag must point to an attribute that exists within the exon lines (e.g., gene_id).

### Gene counts lower than expected across all samples

If overall expression levels appear systematically reduced, this often indicates a structural issue in counting.

- Choosing the wrong feature type (*-t*) leads to underestimation of gene expression. While, as mentioned above, *-t exon* is the standard in most RNA-seq pipelines, different experimental aims require a specific selection. For example, if the goal is to detect unspliced transcripts, *-t gene* should be used instead of *-t exon*.
- Incorrect attribute type (*-g*) selection can fragment counts across multiple identifiers, leading to gene count underestimation.

### Inflated library sizes or unusually high counts 

If total counts per sample are unexpectedly high, especially in paired-end data, counting may be incorrect.

- When analysing data from paired-end sequencing, paired mode (*-p*) should be enabled, or paired reads will be treated as independent single reads, leading to double counting.
- The parameter *--countReadPairs* also needs to be enabled, to ensure that fragments from paired-end reads are counted as single units.

### Loss of reads in specific regions

If specific regions or the overall dataset show reduced counts, filtering parameters such as the following may be too strict:

- Selecting the wrong fragment length thresholds (*-d*, *-D*) can lead to the loss of valid fragments, but also to the inflation of counts if the lower threshold is too small.
- Enabling proper pairing requirement (*-B*) can lead to the loss of valid partially mapped fragments
- While chimeric read exclusion (*-C*) is usually recommended, there are very specific contexts in which doing so might remove potentially biologically relevant reads.

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

Even when alignment or quasi-mapping is successful, quantification with Salmon can produce unexpected or biologically inconsistent expression estimates. In most cases, these issues arise from **mismatches between transcriptome reference, library properties, and quantification parameters**, rather than from the sequencing data itself.

A frequent source of error is **inconsistent transcriptome annotation**. Salmon relies entirely on the provided transcriptome index; if it does not correspond to the same genome build or annotation used elsewhere in the pipeline, reads may be assigned incorrectly or ambiguously, leading to biased transcript and gene-level estimates.

Additionally, Salmon performs transcript-level quantification, which introduces complexities such as multi-mapping reads and isoform ambiguity.

- **Library type specification:** Incorrect library type detection or specification is one of the most common causes of biased quantification.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| `--libType` | A (auto)<br>ISR / SR / IU / etc. | Library type incorrectly specified or auto-detection fails | Misassignment of reads, strand bias, distorted expression estimates |

- **Transcriptome reference:** Salmon depends entirely on the transcriptome index for mapping and quantification.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| Index input | Transcript FASTA | Mismatch with genome build or GTF used downstream | Inconsistent gene-level summarization and missing transcripts |
| Decoy sequences | With/without decoys | Missing decoy-aware index | Increased spurious mapping and inflated expression estimates |

- **Mapping and alignment mode:** Salmon supports both quasi-mapping and alignment-based modes, each with different sensitivities.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| Mapping mode | Quasi-mapping / Alignment-based | Inappropriate mode for data quality or complexity | Reduced accuracy in complex transcriptomes |
| `--validateMappings` | On/off | Disabled selective alignment | Increased false-positive mappings |

- **Bias correction:** Salmon includes advanced models to correct technical biases. Disabling or misusing them can skew results.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| `--seqBias` | On/off | Sequence-specific bias not corrected | Systematic distortion in expression estimates |
| `--gcBias` | On/off | GC bias not modeled | Bias toward GC-rich or GC-poor transcripts |
| `--posBias` | On/off | Positional bias ignored | Inaccurate fragment distribution modeling |

- **Fragment and length modeling:** Salmon models fragment length distributions, which are critical for accurate quantification.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| Fragment length priors | Auto / manual | Incorrect assumptions for fragment size | Biased transcript abundance estimates |
| Paired-end handling | Implicit | Incorrect pairing or inconsistent input | Misestimation of effective lengths |

- **Gene-level summarization:** Salmon outputs transcript-level estimates; gene-level counts require aggregation (e.g., tximport).

| Step | Tool | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| Transcript → gene | tximport / custom mapping | Incorrect or inconsistent transcript-to-gene mapping | Fragmented or incorrect gene-level counts |

- **Quick diagnostic guide:**

| Symptom | Likely cause | Parameter(s) to check |
| :--- | :--- | :--- |
| Strong strand bias or unexpected antisense signal | Incorrect library type | `--libType` |
| Expression detected in unexpected transcripts | Poor transcriptome reference or missing decoys | Index, decoy setup |
| Inflated expression estimates or noisy background | Disabled selective alignment | `--validateMappings` |
| Systematic bias across GC content or transcript length | Bias correction disabled | `--seqBias`, `--gcBias`, `--posBias` |
| Discrepancies between Salmon and featureCounts | Transcript vs gene-level differences or annotation mismatch | Index, tx2gene mapping |
| Fragment length inconsistencies | Incorrect pairing or fragment modeling | Input format, library type |
