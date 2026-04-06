# Post-Alignment Processing

Once the aligner has generated a BAM file, the data is still "raw." It contains biological noise, technical noise, and mapping artifacts. This folder covers the steps required to "clean" the BAM before it is used for downstream analysis

## Blacklist Regions and Non-Canonical Chromosomes

### **Blacklist regions**

Blacklist regions are specific parts of the reference genome that systematically produce artificial high signals in sequencing-based assays — even when there’s no biological reason for enrichment. These mainly include repetitive regions (satelites, rDNA, telomeres, centromeres, etc). Mapping of these regions will give raise to many reads, just because of the high number of repeats, suggesting that there is a real effect when in fact there isn't. ENCODE has a blacklist bed compiled from previous experiments that can be used to exclude these reads from the samples. These are specific to the genome version used in the mapping, so a mismatch will result in the removal of the wrong sequences. The removal of blacklist regions is standard in most NGS pipelines.

### **Non-canonical chromosomes**

The removal of extra sequences not found inside the standard chromosomes 1-22 and X-Y, are usually filtered out. These include the following:

-	**Unplaced contigs** (sequences that are known to be present in the genome, but not precisely where) and **unlocalized contigs** (sequences that are know to be present in a specific chromosome, but their localization inside that chromosome hasn´t been defined). They are leftovers from incomplete assembly — especially in repetitive or highly variable regions (like centromeres, telomeres, immune loci), and can inflate reads artificially.
-	**Alternate haplotypes:** reference genomes include various haplotypes of some genomic regions to represent high variability in structure or sequence. However, small reads can map to several of these haplotypes, making it look as if many reads were detected on the same region (artificial enrichment)
-	**Decoy sequences:** reads that don’t map anywhere in the genome can be captured by contaminant vectors, bacterial DNA or repetitive sequences, generating confusing results. To avoid this, reference genomes contain “decoy” sequences, extra fragments that soak up multi-mapping reads. The reads that map to these regions are not biologically meaningful so they are dropped in most cases.

### The mitochondrial exception

While in other techniques, such as CUT&RUN and ChIP-seq, reads mapping to mitochondrial DNA are removed, since usually the aim is to exclusively study nuclear DNA. However, in ATAC-seq, mitochondrial DNA is normally kept for QC, because being “naked” DNA (no nucleosomes), it's highly targetable by transposases and therefore can serve as a good measurement of the sample’s quality. A high number of mtDNA reads (>50%) means that mitochondria were ruptured and/or the nuclear isolation wasn't performed correctly during sample prep. To avoid this, Omni-ATAC protocol, which uses detergents to specifically wash away mitochondria before transposition, can be used.
RNA-seq, on the other hand keeps both mtDNA and contigs, since an increase in the expression of these unlocalized regions can also be biologically meaningful.

**Note:** For all tecnniques, removing alt haplotypes can technically erase biologically relevant differences — especially in highly polymorphic regions.
We accept that tradeoff because most analysis frameworks aren’t designed for multi-haplotype coordinates.
If the protein is suspected to bind in such regions, the right move is:

1.	keep haplotype contigs,
2.	verify mapping quality in those regions,
3.	and report those peaks separately as haplotype-specific binding.

### What to keep?

| Assay | mtDNA | Haplotypes | Reasoning |
|-------|-------|------------|-----------|
| RNA-seq | Keep | Keep | Expression in these regions can be biologically meaningful |
| ATA-seq | Optional | Drop | mtDNA is used for QC, but then often removed to save budget |
| ChIP/CUT&RUN | Drop | Drop | We only want high-confidence nuclear binding peaks |

## Duplicate Removal

Depending on the technique, the presence of duplicated reads can be a red flag. Duplicated reads are reads that map to the exact same sequence, from start to finish. This usually happens when the PCR amplification step in the library prep generates more copies of a specific fragment (**PCR duplicates**), since random fragmentation is very unlikely to generate reads that start (and sometimes end) at the exact same position. Therefore, these fragments weren´t more present in the original sample, they were just overamplified and are thereby going to generate more reads. 

In RNA-seq it is normal to have many duplicates, since some genes are expressed more than others, and the more transcripts, the more cDNA generated, as opposed to, ATAC-seq, where duplicates are always removed, because since the Tn5 inserts randomly, it is statistically impossible for it to cut at the exact same two genomic coordinates twice in a small cell population. In the case of CUT%RUN/ChIP, suplicates are marked, but they need to be looked into before removal: very narrow peak (like a transcription factor), might give natural duplicates. In this case, some researchers keep them, but the standard conservative pipeline removes them to avoid PCR bias.

Other type of duplicates are the so-called **optical duplicates**. These happen when the sequencer’s camera misidentifies a single cluster as two separate ones (usually because the flow cell is too crowded/overloaded).

The most commonly used tools for duplicate handling are Picard MarkDuplicates and samtools markdup. They group reads/pairs by alignment positions; the read with the highest quality (sum of base qualities) is kept; others are flagged as duplicates. It is important that these duplicates are just flagged and not deleted, since they are used to calculate the QC metric “duplication rate”. Downstream tools are told to ignore these flagged sequences.

Picard actually tries to distinguish between PCR duplicates and Optical duplicates, and the measures to be taken vary depending on the result. If the number of PCR replicates is too high, then the number of PCR cycles needs to be reduced. On the other hand, if the optical duplicates are particularily high, then we need to load less library into the sequencer.

## Spike-in Correction

A spike-in refers to a known quantity of external (exogenous) DNA or RNA that is added (“spiked in”) in a known concentration to a sample before processing. Spike-ins act as internal control or reference, allowing to distinguish biological differences from technical noise. Because it is subjected to the same procedures as the material of interest, it can be used as a normalization step at the end of the process.

While some tools, like DESeq2, perform their own normalization, they assume that the overall transcriptome is unchanged in all samples, and that only differences in certain set of genes are expected. However, some treatments can affect the whole transcriptome, or different cell types can contain different amounts of RNA. In these cases, we need spike-in for the normalization.

The way it works is, a ratio of spike-in reads vs spike-in reads from a reference sample (this can be a control or an average of the spike-in reads for all samples) is calculated. If a sample has fewer endogenous reads per spike-in, that means less material entered the library — but the spike-in alone can’t tell you why. In this scenario, the data would just be normalized to library size, correcting as if a step between library prep and mapping had gone wrong. With spike-in, however, since the added DNA/RNA is subjected to the same sequencing steps and the concentration is equal for all samples, if one of the samples has less read for the DNA/RNA but the same spike-in reads, this will mean that the sample contained less material for whatever reason, allowing to rule out technical issues.
Importantly, this requires that the reads are aligned to both the genome of the organism of interest, and the one used for spike-in.
