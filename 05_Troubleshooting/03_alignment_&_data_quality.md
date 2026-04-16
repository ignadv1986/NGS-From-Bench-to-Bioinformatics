# Mapping Troubleshooting

After quality control, the next critical step in NGS analysis is read alignment (mapping) to a reference genome or transcriptome. Mapping metrics provide insight into how well reads align and can reveal issues related to sample quality, library preparation, contamination, or reference selection.

While, for clarity and organisation, they are contemplated as different steps in this repository, it is important to note that usually both sequencing QC and mapping QC metrics need to be considered in order to properly troubleshoot an experiment.

These metrics are derived directly from the alignment process and reflect how the aligner was able to place reads onto the reference genome under the chosen parameters.

## Mapping Rate

The acceptable % of properly mapped reads varies greatly between different techniques:

<br>

<div align="center">

| Technique	| Aceptable (%)	| Excellent (%)	|
| :--- | :--- | :--- |
| RNA-seq |	>80	| >90 |
| DNA-seq |	>85	| >95	|
| ATAC-seq | >70	| >85	|
| CUT&RUN |	>70	| >80	|

</div>

<br>

When high quality reads show a very low % of mapped reads (10-30%), this strongly suggests a wrong reference genome selection (different species). While not as severe as a species mismatch, choosing a wrong or incomplete version of a reference genome, also leads to a decrease in the % of mapped reads. This can for example happen when mapping RNA-seq data to a full genome with a non-splice-aware aligner (see the [aligners](../02_Mapping_&_Alignment/02_aligners.md) section of this repository for more details).

The presence of contamination in the samples also leads to a decrease in the % of mapped reads (30-70%), since they won't align correctly with the samples's reference genome. As mentioned in the previous section, these foreign reads can usually be spotted by checking the "per sequence GC content" metric of FastQC, where they leade to the formation of "humps" in the expected distribution. Reads mapping to non-target genomes can be filtered out to prevent interference with downstream analysis but, when their % is too high, may have consumed a significant fraction of sequencing capacity, hindering result interpretation. This is why it is important to estimate their percentage with tools like [FastQ Screen](https://www.bioinformatics.babraham.ac.uk/projects/fastq_screen/) or [Kraken2](https://github.com/DerrickWood/kraken2) to assess their impact on the experiment.

The presence of low quality reads (Q score < 28) negatively affects mapping rates. Because the quality per base varies between positions (5' ends due to random hexamer priming, 3' end bias, etc), it is critical that the mapping is performed on already trimmed reads. If, after trimming, the reads still show low Q scores, then the % of mapped reads will drop significantly.

Finally, adapter contamination will also cause a decrease on this metric, since adapters themselves cannot align with the reference genome. However, because they are already detected in the FastQC step of the pipeline, and trimmed away with fastp/cutadapt, adapters are rarely the main cause of a major mapping drop.

## % of Uniquely- Vs Multi-mapped Reads

When mapping reads to a reference genome, a high % of uniquely mapped and a low % of multi mapped reads is highly desirable, as it greatly increases the confidence in their proper alignment. Nevertheless, it is important to know that **certain level of multi-mapping is expected**, due to the presence of repetitive elements in the genome, and such level greatly varies depending on how well the used technique catches these elements.

Additionally, other factors can contribute to an increase in the % of multi-mapped reads:

- **Shorter reads** are less likely to uniquely match a single genomic location. In this case, the overall mapping rate may remain high, but the proportion of uniquely mapped reads decreases
- **Aligner parameters** also play an important role. Permissive settings that allow multiple alignments per read or high mismatch thresholds can artificially inflate the proportion of multi-mapped reads. Therefore, it is important to verify that alignment parameters are appropriate for the experimental design.
- An increase in multi-mapped reads can also result from properties of the **reference genome** itself. Genomic regions with high sequence similarity (e.g., gene families, paralogous genes, or repetitive elements) make it difficult for the aligner to assign reads to a single unique location.

Finally, it is important to note that contamination typically leads to an increase in unmapped reads rather than multi-mapped ones, unless the contaminant sequences share high similarity with the reference genome.

How multi-mapped reads are handled highly depends on the context (mainly used technique and aim of the experiment):

- A common approach when handling multi-mapped reads is to retain only uniquely mapped reads and discard those that align to multiple locations. This simplifies downstream analyses and ensures high confidence in read placement, which is particularly important in applications such as variant calling or strict peak detection. However, this comes at the cost of losing information, especially in repetitive regions or among gene families with high sequence similarity, potentially introducing biases in the results.
- Alternatively, all alignments reported by the aligner can be retained. This preserves the full set of sequencing data and can be useful in analyses focused on repetitive elements or in exploratory settings. However, because the true origin of multi-mapped reads is uncertain, this approach can introduce ambiguity and artificially inflate signal in downstream analyses if not handled carefully.
- A third strategy, commonly used in RNA-seq, is to assign multi-mapped reads probabilistically. In this case, statistical models (like the one used by Salmon) are used to distribute reads across their possible genomic or transcriptomic origins. This approach allows the use of all available data while reducing some of the biases introduced by strict filtering, although it relies on model assumptions and is less straightforward to interpret.

## Integrated Interpretation of Mapping Metrics

<br>

<div align="center">

| Mapping Rate	| Multi-Mapping | Duplication | Interpretation	| Likely Cause	|
| :--- | :--- | :--- | :--- | :--- |
| High |	High | Normal | Reads align well but not uniquely	| Repetitive genome, short reads, permissive alignment settings |
| High | Normal | High | Good alignment, low library complexity	| PCR overamplification, low input material, saturation	|
| Low | Normal | Normal | Reads are high quality but fail alignment	| Wrong reference genome, contamination, annotation mismatch	|

</div>

<br>



