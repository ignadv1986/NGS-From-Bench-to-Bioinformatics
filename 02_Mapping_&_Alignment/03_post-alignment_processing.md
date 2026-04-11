# Post-Alignment Processing

Once the aligner has generated the BAM files, the data is still not ready for downstream processing. It contains biological and technical noise, as well as mapping artifacts. This section covers the steps required to clean the BAM files before they are used for downstream analysis.

## Blacklist Regions and Non-Canonical Chromosomes

### **Blacklist regions**

Blacklist regions are specific parts of the reference genome that systematically produce artificial high signals in sequencing-based assays, even when there’s no biological reason for enrichment. These mainly include **repetitive regions** (satelites, rDNA, telomeres, centromeres, etc). Mapping of these regions will give raise to many reads, just because of the high number of repeats, suggesting that there is a real effect when in fact there isn't. ENCODE has blacklist BED files (such as the V2 blacklists for hg38) compiled from previous experiments that can be used to exclude these reads from the samples. These are specific to the genome version used in the mapping, so a mismatch will result in the removal of the wrong sequences. The removal of blacklist regions is standard in most NGS pipelines.

### **Non-canonical chromosomes**

Extra sequences not found inside the standard chromosomes 1-22 and X-Y, are usually filtered out. These include the following:

-	**Unplaced contigs** (sequences that are known to be present in the genome, but not precisely where) and **unlocalized contigs** (sequences that are know to be present in a specific chromosome, but their localization inside that chromosome hasn´t been defined). They are leftovers from incomplete assembly — especially in repetitive or highly variable regions (like centromeres, telomeres, immune loci), and can inflate reads artificially
-	**Alternate haplotypes:** reference genomes include various haplotypes of some genomic regions to represent high variability in structure or sequence. However, small reads can map to several of these haplotypes, making it look as if many reads were detected on the same region (artificial enrichment)
-	**Decoy sequences:** reads that don’t map anywhere in the genome can be captured by contaminant vectors, bacterial DNA or repetitive sequences, generating confusing results. To avoid this, reference genomes contain decoy sequences, extra fragments that soak up multi-mapping reads. The reads that map to these regions are not biologically meaningful so they are dropped in most cases

**Note:** For all tecnniques, removing alt haplotypes can technically erase biologically relevant differences — especially in highly polymorphic regions.
This trade-off is accepted because most analysis frameworks aren’t designed for multi-haplotype coordinates.
If the protein is suspected to bind in such regions, the right move is tokeep haplotype contigs, verify mapping quality in those regions, and report those peaks separately as haplotype-specific binding

### The mitochondrial exception

In most techniques, where the aim is to exclusively study nuclear genes, reads mapping to mitochondrial DNA are removed. However, in ATAC-seq, mitochondrial DNA is normally kept for QC, because being naked DNA (no nucleosomes), it's highly targetable by transposases and therefore can serve as a good measurement of the sample's quality. A high number of mtDNA reads (>50%) means that mitochondria were ruptured and/or the nuclear isolation wasn't performed correctly during sample prep. To avoid this, Omni-ATAC protocol, which uses detergents to specifically wash away mitochondria before transposition, can be used. Importantly, mtDNA is only used as a measure of sample prep QC, but it is removed before downstream analysis.
RNA-seq, on the other hand keeps both mtDNA and contigs, since an increase in the expression of these unlocalized regions can also be biologically meaningful.

### What to keep?

<br>

<div align="center">

| Assay | mtDNA | Haplotypes | Reasoning |
| :--- | :--- | :--- | :--- |
| RNA-seq | Keep | Drop | Expression in these regions can be biologically meaningful |
| ATA-seq | Keep for QC, drop afterwards | Drop | mtDNA is used for QC, but then often removed to save budget |
| ChIP/CUT&RUN | Drop | Drop | We only want high-confidence nuclear binding peaks |

</div>

<br>

## Duplicate Removal

Depending on the technique, the presence of duplicated reads can be a red flag. Duplicated reads are reads that map to the exact same sequence, from start to finish. Since random fragmentation is very unlikely to generate reads that start (and sometimes end) at the exact same position (especially when using paired-end sequencing) duplicated reads are usually generated when the PCR amplification step in the library prep amplifies more copies of a specific fragment (**PCR duplicates**). Therefore, these fragments weren't more present in the original sample, they were just overamplified and are thereby going to generate more reads. 

In RNA-seq it is normal to have many duplicates, since some genes are expressed more than others (the more transcripts, the more cDNA generated). In ATAC-seq, on the other hand, because the Tn5 inserts randomly, it is statistically impossible for it to cut at the exact same two genomic coordinates twice in a small cell population, and therefore duplicates are always removed. In the case of CUT&RUN/ChIP, duplicates are marked, but they need to be looked into before removal: very narrow peaks (like a transcription factor), might give natural duplicates. In this case, some researchers keep them, but the standard conservative pipeline removes them to avoid PCR bias.

Other type of duplicates are the so-called **optical duplicates**. These happen when the sequencer’s camera misidentifies a single cluster as two separate ones (usually because the flow cell is too crowded/overloaded). To avoid this, it is important to load the right amount of DNA into the sequencer, which is highly dependent on the the type of sequencer used. It is worth mentioning that, in patterned flow cells (NovaSeq/HiSeq 4000), an optical duplicate is often actually a **pad-hop duplicate**. This happens during ExAmp when a DNA fragment escaping one nanowell seeds a neighboring empty well.

The most commonly used tools for duplicate handling are [Picard MarkDuplicates](https://gatk.broadinstitute.org/hc/en-us/articles/360037052812-MarkDuplicates-Picard) and [samtools markdup](https://www.htslib.org/doc/samtools-markdup.html). They group reads/pairs by alignment positions; the read with the highest quality (sum of base qualities) is kept, while others are flagged as duplicates. It is important that these duplicates are just flagged and not deleted, since they are used to calculate the QC metric “duplication rate”: standard tools (Picard/Samtools) apply the **0x400 (1024) SAM flag to duplicate**s, and most downstream analysis tools are configured to recognize this flag and skip these reads automatically.

Picard actually tries to distinguish between PCR duplicates and optical duplicates, and the measures to be taken vary depending on the result. If the number of PCR replicates is too high, then the number of PCR cycles needs to be reduced. On the other hand, if the optical duplicates are particularily high, then we need to load less library into the sequencer.

## Spike-in Correction

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
  <em>Workflow and Logic of Spike-in Normalization in CUT&RUN</em>
</div>

<br>

**Note:** If a spike-in signal is visible on the TapeStation, the spike-in-to-target ratio is too high. This swamping will necessitate significantly higher sequencing depth to recover sufficient target reads for downstream analysis.
