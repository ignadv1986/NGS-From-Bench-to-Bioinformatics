# Sequencing Run & Quality Issues

This section covers issues detected immediately after FASTQ file generation, during primary quality assessment.

**Note:** Before FASTQ file generation, the sequencer performs a **primary analysis** of the data generated during the run, providing **.interop** format files. These files can be inspected using vendor-specific tools, allowing for detailed diagnosis of sequencing instrument failures (e.g. cluster generation, fluidics, imaging issues). Such metrics are outside the scope of this repository and therefore won't be covered here.

## Low Quality Scores

The first step when assessing the quality of the FASTQ files is looking at the Phred scores. Ideally, all samples should have a similar per sequence quality score, and a stable per base sequence quality thoughout the reads.

A **low per base sequence quality in one of the samples** is usually a sign of poor sample/library preparation. However, if this issue is present **across all samples**, a sequencing/run-level issue cannot be discarded and the primary analysis performed by the sequencer needs to be checked. In these cases, because there is a widespread below-threshold Q-score, aggresive trimming will not solve the issue and it is therefore heavily discouraged. If the sequencer shows no sign of failure during the sequencing process, revision of the sample and library preparation, followed by a small pilot experiment to test if the changes made to the protocol solved the issue, is usually recommended.

However, it is important to note that some changes in the quality of specific segments of the reads are normal and should not be misinterpreted as failures:

- **The Q scores are usually lower closer to the end of the reads**, due mainly to the accumulation of phasing and pre-phasing events (the longer the read, the higher the possibility of a phasing event to occur). Other factors, such as reagent depletion over cycles or fluorophore signal decay might also contribute to this drop in quality.
- **In RNA-seq experiments, lower Phred scores at the beginning of reads** can arise from random hexamer priming during reverse transcription. Because Illumina base calling depends on sequence diversity across clusters, reduced diversity in early cycles decreases base-calling confidence and leads to lower Q-scores.

These issues are typically addressed through trimming and should not, on their own, pose a problem for downstream processing.

Additionally, in paired-end sequencing, **R2 reads often show lower overall quality than R1**, especially toward the end of the reads. This is mainly due to inreased accumulation of phasing/pre-phasing events and signal decay over additional sequencing cycles. Consequently, R2 reads usually require heavier trimming than R1. Nevertheless, if the differences are very large, there could be library quality or run issues that need to be investigated.

## Contamination and Sequence Composition Artifacts

The GC distribution shown in the per sequence GC content metric of FastQC is expected to:

- Show a smooth, bell-shaped curve (approximately normal)
- This curve must match the expected GC content in the studied organism

The presence of multiple peaks or a clear deviation from the expected organism's GC profile is indicative of contamination with foreign DNA, either from the sample itself or introduced during library preparation. [FastQ Screen](https://www.bioinformatics.babraham.ac.uk/projects/fastq_screen/) or a k-mer-based tool like [Kraken2](https://github.com/DerrickWood/kraken2) can be used to quantify reads mapping to non-target genomes. A decision must then be made on whether the level of contamination is acceptable, potentially discarding samples if it is too high. The acceptable level of contamination depends on the application and its sensitivity to non-target reads: low levels may be tolerable in exploratory analyses, while even minor contamination can compromise high-resolution tasks such as variant calling. Importantly, if the level of contamination is considered acceptable, reads mapping to non-target genomes can be filtered out to prevent interference in downstream analysis, when appropriate.

An imbalance in A/T vs G/C content across read positions can also be a red flag, although this may reflect either technical bias or true biological signal. This can be observed in the per base sequence content plot when the lines corresponding to each nucleotide are not relatively flat. The presence of "wavy" lines at the 5' end of reads is common and usually caused by random hexamer priming (RNA-seq) or residual adapter/primer effects, while increased A/T content can reflect Tn5 insertion bias (e.g. in ATAC-seq). Alternatively, when an assay is detecting promoter regions, these are usually G/C rich, so an increased level of these bases is normal. These patterns are expected and typically do not require intervention, as they generally do not compromise downstream analysis.

Deviations from uniform base composition that are not caused by true biological signal or a technique-specific bias include:

- Smooth, consistent enrichment in A/T over G/C composition across samples, indicative of library preparation bias (e.g. PCR or fragmentation bias). PCR amplification conditions should be reviewed, and fragmentation methods or settings may need optimization to mitigate this bias.
- Position-specific anomalies, likely arising from sequencing artifacts or residual technical sequences (e.g. adapters). While trimming may suffice to remove these artifacts, it is not a universal solution. If the anomaly is caused by a fundamental lack of library diversity, mapping quality (MAPQ) will remain low regardless of the trimming strategy employed.

## Adapter Contamination

Adapter contamination occurs when sequencing reads extend beyond the DNA insert and into adapter sequences, typically due to short fragment sizes or insufficient size selection during library preparation.

Adapter contamination is typically reported in the FastQC modules "Overrepresented Sequences" and "Adapter Content", and it leads to a sharp drop in quality and sudden changes in composition toward the 3’ end of reads. It can also manifest as reduced mapping rates despite otherwise good overall read quality.

If adapter signal is primarily limited to read ends, trimming with fastp or cutadapt is typically sufficient, followed by a new FastQC run to confirm removal. However, if a large fraction of reads is affected or read length is substantially reduced after trimming, this indicates a library preparation issue and the protocol should be reviewed,

- Fragment size distribution (insert size vs read length)
- Size selection conditions (e.g. SPRI bead ratios)

Importantly, while trimming removes adapter sequences, it does not recover lost insert sequence. If a significant proportion of reads becomes too short after trimming, downstream analyses (e.g. alignment, quantification) may still be compromised.

## Additional QC Signals

These metrics help identify technical issues that are not captured by overall quality scores or composition plots, but can still significantly affect downstream analysis.

### Per Base N Content

The per base N content of the reads should stay close to 0, with a small increase toward the 3' end in some databases. These metrics help identify technical issues not captured by overall quality scores or composition plots, but which can still significantly affect downstream analysis. A per base N content above 1% is generally considered problematic for downstream analysis.

### Sequence Length Distribution

Most sequenced fragments should ideally be of the same size, visualized as a sharp peak at the intended size in this metric. If this is not the case in one or several samples, it is likely due to an issue during library preparation for those samples. When all samples show a consistent peak that does not match the expected size, then the fragment size distribution (read length vs insert size) and the size selection conditions should be reviewed. Importantly, excessive trimming can also alter the sequence length distribution, so it is critical to compare this metric between pre- and post-trimming FastQC runs.

Additionally, if adapter contamination is prominent, an additional peak may appear in the sequence length distribution plot.

### Duplication Levels

The sequence duplication levels metric should show a high proportion of unique reads in most high-complexity libraries such as WGS or exome sequencing. Moderate levels of duplication are expected in RNA-seq due to the presence of highly expressed transcripts, and in targeted assays where enrichment is intentional.

When duplication levels are abnormally high across all samples, this is usually indicative of low library complexity. Common causes include over-amplification during PCR, low input material, or bottlenecks introduced during library preparation.

If high duplication is accompanied by a strong reduction in the proportion of unique reads, the effective sequencing depth is reduced, meaning that additional sequencing will not significantly increase informative content. In these cases, increasing read depth is not recommended unless library complexity issues are addressed.

When duplication is uneven across samples within the same experiment, this can indicate variability in library preparation quality or input material, and those samples should be interpreted with caution in downstream quantitative analyses





