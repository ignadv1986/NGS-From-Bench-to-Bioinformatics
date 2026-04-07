# CUT&RUN

CUT&RUN (Cleavage Under Targets and Release Using Nuclease) is an "in-situ" mapping strategy used to identify the binding sites of DNA-associated proteins, such as transcription factors and histone modifications. Unlike traditional ChIP-seq, which relies on random sonication of cross-linked chromatin, CUT&RUN utilizes a targeted approach within the cellular environment. Cells or nuclei are permeabilized, allowing a specific antibody to bind its target protein. A Protein A-Protein G-Micrococcal Nuclease **(pAG-MNase)** fusion protein then docks onto the antibody. Upon the addition of calcium, the nuclease is activated to precisely cleave the DNA only in the immediate vicinity of the protein-DNA complex.

These small, targeted fragments diffuse out of the nuclei and are collected for sequencing. This method results in extremely low background noise, requires significantly fewer cells (down to 100 cells or fewer in some protocols), and provides high-resolution mapping of protein-DNA interactions. Because the DNA is cleaved specifically at the site of interest, the resulting sequencing data is exceptionally clean, making it a "gold standard" for researchers working with limited sample material or requiring high-precision binding data.

## Fragment Size

While in other techniques, like RNA-seq, the size of the fragments is controlled during library prep, in CUT&RUN, fragments are generated during the protocol by the MNase. Because the nuclease cleaves specifically where the antibody is bound, the length of the resulting DNA is highly informative of the protein’s identity and its "footprint" on the genome. Generally, it is accepted that fragments <120 bp correspond to those bound by transcription factor, while 150 bp and 300 bp correspond to mono- and di-nuclesomes, respectively, indicating that the protein of interest is a histone or a histone binding protein. Fragments of 500 bp or above are usually considered background. Therefore, the SPRI step of the library prep is highly dependent on what is known about the protein of interest. In all cases, a double-sided size selection is recommended: first, a low ratio (~0.6x) is used to remove the big, uninformative pieces of DNA. The beads are discarded and the supernatant is kept. Depending on the type of protein being studied, the second SPRI, isolation step varies:

| Type of protein | Size (after adapter binding) | SPRI ratio | Purpose |
|-----------------|------------------------------|------------|---------|
| Transcription factor | 140-240 bp | 1x-1.2x | Captures small footprints while excluding <120 bp adapter dimers |
| Histone-binding factor | ~270 bp | 0.8×-0.9× | Captures nucleosomal units while aggressively cleaning out TF "noise" and dimers |

Now the DNA of interest will be bound to the beads, and the supernatant can be discarded. After a series of 80% ethanol washes, the purified library is eluted.

When assessing fragment size with the TapeStation, a clean peak at the target size, with no peak at 120 bp, should be observed. The presence of a sharp, dominant peak at 120–130 bp indicates that your SPRI ratio was too high (captured adapter dimers) or that the MNase digestion was insufficient, leaving the adapters with nothing to bind to but themselves.

When the binding nature of the protein is unknown, the goal is to preserve the entire biological spectrum—from tiny footprints to large nucleosomal structures. This is achieved through a 1.8x-2.0x SPRI isolation step. In this scenario, an adapter dimer peak is often unavoidable at the bench. In such cases, if the dimer signal exceeds the biological signal, alternative purification methods—such as automated gel excision (e.g., Pippin Prep) set to a >140 bp collection window—may be required to salvage the library for sequencing.

There are many tools to calculate the fragment size distribution, but [bedtools](https://bedtools.readthedocs.io/en/latest/) or the **bamPEFragmentSize** of [deepTools](https://deeptools.readthedocs.io/en/latest/) are usually the go-to options.

## The IgG Control

Every CUT&RUN experiment must include an IgG (non-specific antibody) control. The IgG sample should have very few mapped reads and a flat, noisy profile in the genome browser. If the IgG condition looks similar to the target antibody (same peaks, same signal), the enrichment is likely just background noise or open chromatin artifacts.

If the IgG shows a massive, high-molecular-weight smear on the TapeStation, it indicates a failure in the wash steps: the MNase wasn't washed away and was allowed to digest the genome globally, which defeats the purpose of the control.

Since the IgG is a low-complexity library—it represents only the rare, non-specific locations where the MNase happened to cut without antibody guidance-it should not be pooled at the same ratio as the actual samples: ff the target is pooled at 4 nM, the IgG should often be pooled at 1 nM or 2 nM. This saves a lot of space on the flow cell that would otherwise be used to generate many duplicates from the unique fragments present in the IgG sample.

## PCR amplification

After the adapters are ligated to the MNase-cut fragments, the library must be amplified to reach the concentration required for sequencing (typically 0.5–10 nM). Most CUT&RUN protocols aim for 12–14 cycles. If the protein is a high-abundance histone mark, 10 cycles may be sufficient. The PCR reaction must use a high-fidelity polymerase and an optimized extension time. If the extension time is too long, the polymerase may favor longer fragments, causing the library to lose the tiny, 50 bp transcription factor footprints that are the hallmark of CUT&RUN.

It is worth noting that is perfectly normal to see no DNA in the IgG sample, since in this sample there was no antibody to guide the MNase. The IgG conditions should never be run for more cycles than the experimental samples. If the IgG is amplified more, the background noise would be artificially inflated, and if it shows no DNA after the same number of cycles as the target, that is actually a sign of a very clean experiment with low non-specific binding.

## Quantification

While PCR amplification exponentially enriches for fragments with dual adapters, fluorometric methods may still over-estimate sequenceable library concentration, due to the presence of non-amplified genomic fragments carried over through the SPRI steps and primer dimers. It is therefore advisable to obtain the molarity through qPCR, unless the TapeStation shows a clear, high-yield peak with no presence of adapter primers. 

## Spike-in Normalization & Scaling Factor

Because CUT&RUN has such a low background signal, traditional normalization methods (like total read count) can be misleading. It is critical to normalize using spike-in DNA (usually E. coli or yeast DNA added during the protocol). For a deeper dive into the theory behind this, see the section on [Spike-in Correction](../02_Mapping_&_Alignment/03_post-alignment_processing.md) of this repository.

In CUT&RUN, we normalize technical variations by calculating a **scaling factor** for each sample:

Scaling Factor = max spike-in reads across all samples / sample spike-in reads

​This normalizes technical variations between samples, since the apike-in  DNA goes through the same processes as the samples themselves. If spike-in fails for a sample (very low counts), scaling may produce extreme inflation and should be excluded.

## Early Visualization

The coverage from each replicate and the merge files can be visualized using the BAM files on [SeqMonk](https://www.bioinformatics.babraham.ac.uk/projects/seqmonk/), [IGV](https://igv.org) or the [USCS browser](https://genome.ucsc.edu). This serves as a QC control, where enrichment at expected regions, replicate similarity and whole genome background can be assessed.

When raw BAM files are viewed, the IgG control may appear to have peaks as tall as the experimental samples. This is caused by an artifact of auto-scaling within the browser. Because the IgG library contains very few reads, the background noise is zoomed in upon by the software to fill the vertical space of the track. Bigwig generation and spike-in normalization must be completed before the true, scaled relationship between the samples can be observed.

## Coverage Files: bedgraph vs bigwig

Once the reads have been mapped and filtered, the next step in a CUT&RUN workflow is generating the **coverage files**. These files translate discrete aligned reads into a continuous signal profile representing the number of fragments overlapping each genomic region.

There are two ways to represent coverage from BAM files: generating **bedGraph** or **BigWig** files. bedGraph are plain text versions, easily readable, while BigWig are a compressed binary version, lighter but not human-readable. In consequence, BigWig files are smaller and easier to read for softwares but, unlike bedGraph, they cannot be edited.

Both bedGraph and BigWig files are usually generated with the **bamcoverage** function of [deepTools](https://deeptools.readthedocs.io/en/latest/). This tool is preferred because it allows the spike-in scaling factor to be applied during the conversion process, thereby producing already normalized bedGraph/bigWig files.

The choice between generating bedGraph or BigWig files is largely dictated by the requirements of the downstream peak-calling software (see next section), but the generation of BigWig files is always recommended to ensure smooth, high-performance visualization in IGV or other genome browsers. It is possible to generate BigWig files from bedGraph with [bedGraphToBigWig](https://anaconda.org/channels/bioconda/packages/ucsc-bedgraphtobigwig/overview). 

## Peak Calling

Following the generation of normalized coverage profiles, the final critical step in the CUT&RUN pipeline is peak calling. This process uses statistical models to identify genomic regions where the density of mapped fragments is significantly higher than the background noise (**peaks**), indicating a **true protein-DNA binding event** or histone modification. The goal of this step is to produce a BED or narrowPeak file containing the coordinates of these high-confidence binding sites for downstream functional analysis.

### bed and narrowPeak files

Peak infomation is stored in tab-delimited files known as **bed**(browser extensible data). They contain the chromosome number and the start and end of the coordinates for the different detected peaks. However, other elements can be attached to them for downstream analysis. Another common format of files used to store peak information is **narrowPeak**. These are basically 10-column bed files, with additional information such as the peak "summit" (the exact point of highest signal), fold-enrichment, and statistical significance (p-values and q-values).

### Peak calling tools

The two main tools used for peak-calling are [MACS3](https://macs3-project.github.io/MACS/) and [SEACR](https://github.com/FredHutch/SEACR). While both can be used for CUT&RUN analysis, **SEACR** (Simple Enrichment Analysis of CUT&RUN) was specifically created for this purpose. Opposed to ChIP-seq, CUT&RUN´s extremely low-background signal, with clear and spiked peaks, makes it perfect for an analysis with a threshold-based algorithm like the one SEACR uses, instead of statistical background models. Importantly, SEACR requires input in the form of bedgraph files and returns bed files, which can later be converted to narrowpeaks, while MACS2 uses BAM files as input and returns narrowpeaks. It is worth mentioning that SEACR can be run in “relaxed” or “stringent” modes, with the latter usually being the preferred option due to the low background typical of CUT&RUN experiments.
**MACS3**, on the other hand is better for the detection of broad histone marks and other “wider” peaks, and is therefore the go-to option when analyzing ChIP-seq data, but also useful in specific CUT&RUN experiments. It takes BAM files as input and returns bed files, so bedGraph generation is normally skipped in a MACS3-based workflow. 

Importanyly, in CUT&RUN, peak callers use the IgG control to establish a baseline. By comparing the experimental sample to the IgG, the algorithm can distinguish between true biological enrichment and technical artifacts or "open chromatin" noise.

### Visualization in genome browsers

As mentioned above, the ideal file format to visualize enrichment is BigWig. Because SEACR provides bedGraph files, converting these to BigWig with the bedGraphToBigWig tool is highly recommended to inspect the data. In the case of MACS3, BigWig files are generated from BAM files with bamcoverage.

It is also good practice to obtain BigWig files from merged replicates (see below). Comparing these tracks to the ones of the individual replicates in a browser allows for a qualitative sanity check to ensure that the merged signal accurately represents the individual replicates and that no unexpected artifacts were introduced during the pooling process.

### Handling of replicates

Unlike algorithms that use statistical background modeling, SEACR does not have a built-in function to handle biological replicates simultaneously. Because it is highly sensitive to coverage depth, a specific workflow must be adopted to ensure the resulting peak set is both reproducible and statistically sound. There are two primary approaches to analyzing experiments with multiple replicates:

- **BAM merging:** BAM files are merged per condition, converted to bedGraph, and processed through SEACR to obtain a consensus peak set. Read counts are then quantified at these peak locations for each individual replicate. While this is often the recommended method for maximizing sensitivity, it requires that all replicates are of similar quality; therefore, a rigorous pre-QC step is mandatory.
- **Intersection:** SEACR is run on each replicate independently. A final peak set is generated by keeping only the peaks present in a specific number of replicates (e.g., 2 out of 3) using a process called intersection. While this method ensures high reproducibility, it can be less powerful for detecting low-signal peaks than the merged approach.

To capture the benefits of both sensitivity and reproducibility, a common approach is to combine these strategies:

- Replicate BAM files are merged for all conditions, converted to bedGraph, and processed through SEACR. (In a MACS3 workflow, individual narrowPeak files are typically merged or pooled).
- Peaks are called in independent replicates. Only those present in at least 2 out of 3 replicates are identified (using **bedtools multiinter**).
- The reproducible peaks from step 2 are intersected with the master set from step 1 (using **bedtools intersect**) to build the **final consensus peak set**.

Why is intersection necessary? When files are merged, subtle differences in the signal shape of individual replicates can cause shifts in peak boundaries. Intersecting ensures that the reported peak width is consistent with the individual replicates (which is likely more accurate) while confirming the peak's presence in the higher-depth merged file. Furthermore, merging can create artifacts where weak signals from multiple replicates combine to "trick" SEACR into calling a peak that wasn't reproducible; intersection effectively prevents these artifacts from being included in the final data.

## Read counting on consensus peaks

After identifying the consensus peaks, the next step is to count the reads on those peaks across all replicates for all conditions. First, all consensus peaks files are merged into a **single unique consensus bed file** that will contain the peaks detected in all conditions. A matrix is then built with these BED and the BigWig files using multiBigWigSummary. Alternatively, if BAM are the preferred format, bedtools coverage or deepTools multicov would be the go-to option. This matrix serves as the foundation for downstream statistical testing and differential enrichment analysis.

## Biological interpretation

Once a consensus peak set has been generated and quantified, the focus shifts from data processing to biological discovery. This involves characterizing the peaks based on their genomic location, associated genes, and DNA sequence properties.

### Comparison of conditions

When multiple experimental conditions were analyzed, the first step to find differences between said conditions is to look for peaks that are exclusive to a specific condition, or peaks that are absent in one but not in the rest. This is achieved with **bedtools intersect**.

### Region of interest (ROI) visualization

If a signal acroos a specific region of interest needs to be analyzed, first a A BED file of coordinates must be generated. These can be specific genes of interest from Ensembl/NCBI or borad categories such as "all promoters" (±2 kb from the transcription start site (TSS)). Once the BED file is generated, **deepTools computeMatrix** can be used on the BigWig files to map the signal density acroos these regions. The output is visualized via **deepTools plotHeatmap** or **deepTools plotProfile**.

### Peak annotation & functional analysis

After identifying peaks, these can be annotated to genomic features, such as promoters, enhancers or intergenic regions with the R/Bioconductor package **[ChIP-Seeker](https://bioconductor.org/packages/release/bioc/html/ChIPseeker.html)**.

This allows for functional enrichment analysis (GO Terms/KEGG Pathways) to see if the identified peaks are associated with specific biological processes.

### Motif enrichment analysis

Finally, the enrichment of specific motifs, such as known transcription factor binding sites can be analyzed. The table below summarizes the main tools used for motif enrichment and their goals:

| Tool | Focus | Best for |
|------|-------|----------|
| [HOMER](http://homer.ucsd.edu/homer/) | Known motifs + simple *de novo* | Quick, reliable enrichment of established motifs |
| [MEME Suite](https://meme-suite.org/meme/) | Advanced *de novo* discovery | Identifying novel or complex motifs with high statistical rigor |
| [TOBIAS](https://github.com/loosolab/TOBIAS) | TF footprinting / activity | Analyzing differential TF occupancy within open chromatin (often paired with ATAC-seq) |





