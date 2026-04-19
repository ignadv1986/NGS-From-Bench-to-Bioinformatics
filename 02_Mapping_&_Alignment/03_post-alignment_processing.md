# Post-Alignment Processing

Once the aligner has generated the BAM files, the data is still not ready for downstream processing. These BAM files are not sorted or indexed, and contain biological and technical noise, as well as mapping artifacts. This section covers the steps required to clean the BAM files before they are used for downstream analysis.

## BAM sorting

By default, most sequencing machines produce data in the order the sequences were read, which is effectively random. In order to effectively perform downstream analysis, BAM files need to be sorted by **genomic coordinates** with the *sort* command of the [samtools](https://www.htslib.org) package. For consistency, sorted files are usually created with the name sample.sorted.bam.

## Duplicate Marking

Depending on the technique, the presence of duplicated reads can be a red flag. Duplicated reads are reads that map to the exact same sequence, from start to finish. Since random fragmentation is very unlikely to generate reads that start (and sometimes end) at the exact same position (especially when using paired-end sequencing) duplicated reads are usually generated when the PCR amplification step in the library prep amplifies more copies of a specific fragment (**PCR duplicates**). Therefore, these fragments weren't more present in the original sample, they were just overamplified and are thereby going to generate more reads. In most pipelines, duplicate reads are identified and **flagged (marked)** rather than physically removed from the BAM file.

In RNA-seq it is normal to have many duplicates, since some genes are expressed more than others (the more transcripts, the more cDNA generated). In ATAC-seq, on the other hand, because the Tn5 inserts randomly, duplicates are assumed to be technical and therefore marked in standard pipelines. In the case of CUT&RUN/ChIP, duplicates are marked, but they need to be looked into with caution: very narrow peaks (like a transcription factor), might give natural duplicates. In this case, some researchers keep them in downstream analysis, but the standard conservative pipeline removes them to avoid PCR bias.

Other type of duplicates are the so-called **optical duplicates**. These happen when the sequencer’s camera misidentifies a single cluster as two separate ones (usually because the flow cell is too crowded/overloaded). To avoid this, it is important to load the right amount of DNA into the sequencer, which is highly dependent on the type of sequencer used. It is worth mentioning that, in patterned flow cells (NovaSeq/HiSeq 4000), an optical duplicate is often actually a **pad-hop duplicate**. This happens during ExAmp when a DNA fragment escaping one nanowell seeds a neighboring empty well.

The most commonly used tools for duplicate handling are [Picard MarkDuplicates](https://gatk.broadinstitute.org/hc/en-us/articles/360037052812-MarkDuplicates-Picard) and [samtools markdup](https://www.htslib.org/doc/samtools-markdup.html). They group reads/pairs by alignment positions; the read with the highest quality (sum of base qualities) is kept, while others are flagged as duplicates. It is important that these duplicates are just flagged and not deleted, since they are used to calculate the QC metric “duplication rate”: standard tools (Picard/Samtools) apply the **`0x400` (1024) SAM flag to duplicates**, and most downstream analysis tools are configured to recognize this flag and skip these reads automatically.

Picard actually tries to distinguish between PCR duplicates and optical duplicates, and the measures to be taken vary depending on the result. If the number of PCR replicates is too high, then the number of PCR cycles needs to be reduced. On the other hand, if the optical duplicates are particularly high, a lower library concentration should be loaded into the sequencer.

Some pipelines explicitly remove duplicate reads from the BAM files after marking (e.g., in ATAC-seq or ChIP-seq workflows). In these cases, because reads are deleted, the coordinate order of the BAM file may be disrupted and re-sorting is required. The main reasons why some pipelines remove marked duplicates, even though in theory the flag makes them invisible for downstream tools are:

- **File size:** Removal of duplicates reduces the effective size of the BAM files, saving space and speeding up downstream analysis.
- **Tool compatibility:** While some tools recognize the flag imposed on duplicated reads by samtools or Picard, others require the user to set specific parameters. Removing duplicates makes the analysis "safer".
 
## Blacklist Regions and Non-Canonical Chromosomes

### **Blacklist regions**

Blacklist regions are specific parts of the reference genome that systematically produce artificial high signals in sequencing-based assays, even when there’s no biological reason for enrichment. These mainly include **repetitive regions** (satellites, rDNA, telomeres, centromeres, etc). Mapping of these regions will give rise to many reads, just because of the high number of repeats, suggesting that there is a real effect when in fact there isn't. ENCODE has blacklist BED files (such as the V2 blacklists for hg38) compiled from previous experiments that can be used to exclude these reads from the samples. These are specific to the genome version used in the mapping, so a mismatch will result in the removal of the wrong sequences. The removal of blacklist regions is standard in most NGS pipelines.

### **Non-canonical chromosomes**

Extra sequences not found inside the standard chromosomes 1-22 and X-Y, are usually filtered out. These include the following:

-	**Unplaced contigs** (sequences that are known to be present in the genome, but not precisely where) and **unlocalized contigs** (sequences that are know to be present in a specific chromosome, but their localization inside that chromosome hasn't been defined). They are leftovers from incomplete assembly — especially in repetitive or highly variable regions (like centromeres, telomeres, immune loci), and can inflate reads artificially
-	**Alternate haplotypes:** reference genomes include various haplotypes of some genomic regions to represent high variability in structure or sequence. However, small reads can map to several of these haplotypes, making it look as if many reads were detected on the same region (artificial enrichment)
-	**Decoy sequences:** reads that don’t map anywhere in the genome can be captured by contaminant vectors, bacterial DNA or repetitive sequences, generating confusing results. To avoid this, reference genomes contain decoy sequences, extra fragments that soak up multi-mapping reads. The reads that map to these regions are not biologically meaningful so they are dropped in most cases

**Note:** For all techniques, removing alt haplotypes can technically erase biologically relevant differences — especially in highly polymorphic regions.
This trade-off is accepted because most analysis frameworks aren’t designed for multi-haplotype coordinates.
If the protein is suspected to bind in such regions, the right move is to keep haplotype contigs, verify mapping quality in those regions, and report those peaks separately as haplotype-specific binding

### The mitochondrial exception

In most techniques, where the aim is to exclusively study nuclear genes, reads mapping to mitochondrial DNA are removed. However, in ATAC-seq, mitochondrial DNA is normally kept for QC, because being naked DNA (no nucleosomes), it's highly targetable by transposases and therefore can serve as a good measurement of the sample's quality. A high number of mtDNA reads (>50%) means that mitochondria were ruptured and/or the nuclear isolation wasn't performed correctly during sample prep. To avoid this, Omni-ATAC protocol, which uses detergents to specifically wash away mitochondria before transposition, can be used. Importantly, mtDNA is only used as a measure of sample prep QC, but it is removed before downstream analysis.
RNA-seq, on the other hand keeps both mtDNA and contigs, since an increase in the expression of these unlocalized regions can also be biologically meaningful.

### What to keep?

<br>

<div align="center">

| Assay | mtDNA | Haplotypes | Reasoning |
| :--- | :--- | :--- | :--- |
| RNA-seq | Keep | Drop | Expression in these regions can be biologically meaningful |
| ATAC-seq | Keep for QC, drop afterwards | Drop | mtDNA is used for QC, but then often removed to save budget |
| ChIP/CUT&RUN | Drop | Drop | We only want high-confidence nuclear binding peaks |

</div>

<br>

## Final Sorting and Indexing

The previous steps disrupt the coordinate order of the BAM files and therefore they need to be sorted again. To avoid the generation of multiple intermediate files, that would otherwise take up a lot of space and makes file tracking difficult, blacklist region and chromosome/haplotype removal are often combined into one step. Additionally, most downstream analysis tools require the BAM files to be **indexed**, which is usually performed with the *index* command of samtools. This generates a **.bai** version of the bam files, and it is good practice to keep them in the same folder to facilitate downstream analysis. Importantly, any modification to a BAM file invalidates its index, so indexing must be performed after the final processing step.

While the pipeline here described (sorting -> duplicate marking -> filtering -> sorting and indexing) is standard in most cases, some pipelines perform filtering before duplication removal for computational efficiency, at the cost of potentially altering duplicate detection.

At the end of this process, the result is a **coordinate-sorted, indexed, and filtered BAM file** suitable for downstream analyses such as peak calling, visualization, or quantification.
