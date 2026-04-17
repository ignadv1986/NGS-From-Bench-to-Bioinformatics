# Spike-in Control

A spike-in refers to a **known quantity of external (exogenous) DNA or RNA** that is added (“spiked in”) in a known concentration to a sample before processing. Spike-in acts as an internal control or reference, allowing to distinguish biological differences from technical noise. Because **it is subjected to the same procedures as the material of interest**, it can be used as a normalization step at the end of the process.

While some tools, like DESeq2, perform their own normalization, they assume that the overall transcriptome is unchanged in all samples, and that only differences in certain set of genes are expected. However, some treatments can affect the whole transcriptome, or different cell types can contain different amounts of RNA. In these cases, we need spike-in for the normalization.

The way it works is, a ratio of spike-in reads vs spike-in reads from a reference sample (this can be a control or an average of the spike-in reads for all samples) is calculated. If a sample has fewer endogenous reads per spike-in, that means less material entered the library. In this scenario, the data would just be normalized to library size, correcting as if a step between library prep and mapping had gone wrong. With spike-in, however, since the added DNA/RNA is subjected to the same sequencing steps and the concentration is equal for all samples, if one of the samples has less reads for the DNA/RNA but the same spike-in reads, this will mean that the sample contained less material for whatever reason, allowing to rule out technical issues.
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

**Note:** If a spike-in signal is visible on the TapeStation, the spike-in-to-target ratio is too high. This swamping will necessitate significantly higher sequencing depth to recover sufficient target reads for downstream analysis.
