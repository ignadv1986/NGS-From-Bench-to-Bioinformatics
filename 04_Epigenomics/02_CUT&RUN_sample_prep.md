# CUT&RUN: Technical Details on Sample Preparation

CUT&RUN (Cleavage Under Targets and Release Using Nuclease) is an "in-situ" mapping strategy used to identify the binding sites of DNA-associated proteins, such as transcription factors and histone modifications. Unlike traditional ChIP-seq, which relies on random sonication of cross-linked chromatin, CUT&RUN utilizes a targeted approach within the cellular environment. Cells or nuclei are permeabilized, allowing a specific antibody to bind its target protein. A Protein A-Protein G-Micrococcal Nuclease **(pAG-MNase)** fusion protein then docks onto the antibody. Upon the addition of calcium, the nuclease is activated to precisely cleave the DNA only in the immediate vicinity of the protein-DNA complex.

These small, targeted fragments diffuse out of the nuclei and are collected for sequencing. This method results in extremely low background noise, requires significantly fewer cells (down to 100 cells or fewer in some protocols), and provides high-resolution mapping of protein-DNA interactions. Because the DNA is cleaved specifically at the site of interest, the resulting sequencing data is exceptionally clean, making it a "gold standard" for researchers working with limited sample material or requiring high-precision binding data.

<br>

<div align="center">
  
  <img src="../Figures/CUT&RUN.png" width="600">
  <br>
  <em>"Visual Representation of the CUT&RUN Sequencing Work-Flow" by Mannan369 (2018), Wikimedia Commons. 
Licensed under Creative Commons Attribution-Share Alike 4.0 International (https://creativecommons.org/licenses/by-sa/4.0/deed.en).
Source: (https://commons.wikimedia.org/wiki/File:CUT%26RUN_Protocol.tif)</em> 
</div>

<br>

## Fragment Size

While in other techniques, like RNA-seq, the size of the fragments is controlled during library prep, in CUT&RUN, fragments are generated during the protocol by the MNase. Because the nuclease cleaves specifically where the antibody is bound, the length of the resulting DNA is highly informative of the protein's identity and its "footprint" on the genome. Generally, it is accepted that fragments <120 bp correspond to those bound by a transcription factor, while 150 bp and 300 bp correspond to mono- and di-nucleosomes, respectively, indicating that the protein of interest is a histone or a histone-binding protein. Fragments of 500 bp or above are usually considered background. Therefore, the SPRI step of the library prep is highly dependent on what is known about the protein of interest. In all cases, a double-sided size selection is recommended: first, a low ratio (~0.6x) is used to remove the big, uninformative pieces of DNA. The beads are discarded and the supernatant is kept. Depending on the type of protein being studied, the second SPRI, isolation step varies:

<br>

<div align="center">

| Type of protein | Size (after adapter binding) | SPRI ratio | Purpose |
|-----------------|------------------------------|------------|---------|
| Transcription factor | 140-240 bp | 1x-1.2x | Captures small footprints while excluding <120 bp adapter dimers |
| Histone-binding factor | ~270 bp | 0.8x-0.9x | Captures nucleosomal units while aggressively cleaning out TF noise and dimers |

</div>

<br>

Now the DNA of interest will be bound to the beads, and the supernatant can be discarded. After a series of 80% ethanol washes, the purified library is eluted.

When assessing fragment size with the TapeStation, a clean peak at the target size, with no or a very small peak at 120 bp, should be observed.

**Note:** In CUT&RUN, when the binding nature of the protein is unknown, the goal is to preserve the entire biological spectrum—from tiny footprints to large nucleosomal structures. This is achieved through a 1.8x-2.0x SPRI isolation step. In this scenario, an adapter dimer peak is often unavoidable at the bench. In such cases, if the dimer signal exceeds the biological signal, alternative purification methods—such as automated gel excision (e.g., Pippin Prep) set to a >140 bp collection window—may be required to salvage the library for sequencing.

<br>

<div align="center">
  
  <img src="../Figures/CUT&RUN_fragment_size.png" width="600">
  <br>
  <em>Different possibilities of fragment size distribution in a CUT&RUN experiment</em> 
</div>

<br>

While the TapeStation provides a physical assessment of the library before it hits the flow cell, it is essential to verify the fragment size distribution bioinformatically after sequencing. Tools such as [bedtools](https://bedtools.readthedocs.io/en/latest/) or the [bamPEFragmentSize](https://deeptools.readthedocs.io/en/develop/content/tools/bamPEFragmentSize.html) of deepTools are used on the aligned BAM files to generate a fragment size histogram. This step serves as a final quality control to ensure that the biological signal (e.g., the 140-200 bp transcription factor footprints or ~300 bp nucleosomal peaks) was successfully captured and preserved throughout the sequencing process, and to confirm that the data is not dominated by technical artifacts like adapter dimers.

## The IgG Control

Every CUT&RUN experiment must include an IgG (non-specific antibody) control. The IgG sample should have very few mapped reads and a flat, noisy profile in the genome browser. If the IgG condition looks similar to the target antibody (same peaks, same signal), the enrichment is likely just background noise or open chromatin artifacts.

If the IgG shows a massive, high-molecular-weight smear on the TapeStation, it indicates a failure in the wash steps: the MNase wasn't washed away and was allowed to digest the genome globally, which defeats the purpose of the control.

Since the IgG is a low-complexity library—it represents only the rare, non-specific locations where the MNase happened to cut without antibody guidance-it should not be pooled at the same ratio as the actual samples: if the target is pooled at 4 nM, the IgG should often be pooled at 1 nM or 2 nM. This saves a lot of space on the flow cell that would otherwise be used to generate many duplicates from the unique fragments present in the IgG sample.

## PCR amplification & Handling of Duplicates

After the adapters are ligated to the MNase-cut fragments, the library must be amplified to reach the concentration required for sequencing (typically 0.5–10 nM). Most CUT&RUN protocols aim for 12–14 cycles. If the protein is a high-abundance histone mark, 10 cycles may be sufficient. The PCR reaction must use a high-fidelity polymerase and an optimized extension time. If the extension time is too long, the polymerase may favor longer fragments, causing the library to lose the tiny, 50 bp transcription factor footprints that are the hallmark of CUT&RUN.

A critical decision point in the CUT&RUN bioinformatic pipeline is the management of duplicate reads—identical fragments that map to the same genomic coordinates. While standard ChIP-seq protocols often mandate the removal of duplicates to mitigate PCR bias, CUT&RUN requires a more nuanced approach. Due to the targeted nature of pAG-MNase cleavage and the low starting cell numbers typical of CUT&RUN, multiple cells often yield identical DNA fragments. These are categorized as **biological duplicates** rather than technical artifacts. Indiscriminate removal of these reads can artificially truncate peak signal and reduce the dynamic range of the data, particularly for high-abundance targets like histone marks. Keeping the duplicate reads is recommended when PCR cycles are kept low (12–14 cycles) and the duplication rate remains within a standard range (<30–40%). This preserves the quantitative relationship between binding sites.

It is worth noting that it is perfectly normal to see no DNA in the IgG sample, since in this sample there was no antibody to guide the MNase. The IgG conditions should never be run for more cycles than the experimental samples. If the IgG is amplified more, the background noise would be artificially inflated, and if it shows no DNA after the same number of cycles as the target, that is actually a sign of a very clean experiment with low non-specific binding.

## Quantification

While PCR amplification exponentially enriches for fragments with dual adapters, fluorometric methods may still over-estimate sequenceable library concentration due to the presence of non-amplified genomic fragments carried over through the SPRI steps and primer dimers. It is therefore advisable to obtain the molarity through qPCR, unless the TapeStation shows a clear, high-yield peak with no presence of adapter primers.

## Spike-in Control

A spike-in refers to a **known quantity of external (exogenous) DNA or RNA** that is added (“spiked in”) in a known concentration to a sample before processing. Spike-in acts as an internal control or reference, allowing to distinguish biological differences from technical noise. Because **it is subjected to the same procedures as the material of interest**, it can be used as a normalization step at the end of the process.

While some tools, like DESeq2, perform their own normalization, they assume that the overall transcriptome is unchanged in all samples, and that only differences in certain sets of genes are expected. However, some treatments can affect the whole transcriptome, or different cell types can contain different amounts of RNA. In these cases, we need spike-in for the normalization.

The way it works is, a ratio of spike-in reads vs spike-in reads from a reference sample (this can be a control or an average of the spike-in reads for all samples) is calculated. If a sample has fewer endogenous reads per spike-in, that means less material entered the library. In this scenario, the data would just be normalized to library size, correcting as if a step between library prep and mapping had gone wrong. With spike-in, however, since the added DNA/RNA is subjected to the same sequencing steps and the concentration is equal for all samples, if one of the samples has fewer reads for the DNA/RNA but the same spike-in reads, this will mean that the sample contained less material for whatever reason, allowing to rule out technical issues.
Importantly, this requires that the reads are aligned to both the genome of the organism of interest and the one used for spike-in.

While spike-in DNA is present from the initial stages of the protocol and undergoes the same PCR amplification as the target library, it is typically absent from TapeStation or Bioanalyzer traces. This is due to two main reasons:

- **Concentration:** To prevent the spike-in from dominating sequencing depth, it is added at a concentration designed to constitute only 1–5% of the final library.
- **Distribution:** Specifically for CUT&RUN, unlike the target DNA, which is cleaved at specific intervals by tethered MNase to form distinct peaks, spike-in DNA undergoes non-specific, random cleavage. This results in a broad, low-molarity smear that falls below the limit of fluorescent detection.

<br>

<div align="center">
  <img src="../Figures/spike-in.png" width="700">
  <br>
  <em>Workflow and Logic of Spike-in Normalization</em>
</div>

<br>

**Note:** If a spike-in signal is visible on the TapeStation, the spike-in-to-target ratio is too high. This over-saturation will necessitate significantly higher sequencing depth to recover sufficient target reads for downstream analysis.


