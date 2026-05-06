# Mapping Principles and QC

Once the sequencing data has been cleaned and validated through QC, the next step in the primary analysis pipeline is the **mapping** or **alignment**. This is the process of determining the original coordinate of each DNA fragment by comparing the sequences in the FASTQ files to a known **reference genome**. Choosing the correct reference and the appropriate alignment algorithm is critical, as any errors at this stage will propagate through to the final analysis.

## Reference Genomes

The most widely used versions of human reference genome are GRCh37 and GRCh38, with the latter being more recent and constantly updated. However, if the data or database used in the experiment is still on version GRCh37, then the same reference genome should be used. It is important to note that these are the [Ensembl](https://www.ensembl.org/index.html) reference genomes. Their [UCSC](https://hgdownload.soe.ucsc.edu/downloads.html) counterparts (hg19 for GRCH37 and hg38 for GRCh38) are equally valid but differ in naming and structure. While in the Ensembl genomes chromosomes are named with only numbers, UCSC uses chr1, chr2, etc. Some alignment tools require exact chromosome names, or they will fail or mismatch, so it is essential to choose one format and stick with it for the whole analysis.

•	Ensembl is a genome browser and annotation platform maintained by EMBL–EBI. It provides genome assemblies, gene annotations, variation data, and other reference resources for many species.

•	GENCODE is a specific annotation project that focuses on producing high-quality, manually curated gene models — initially for human and mouse genomes.

GENCODE and Ensembl are now technically the same for human and mouse. Since 2018, Ensembl uses the GENCODE gene set as its default, but the latter includes a lot of long non-coding RNAs (lncRNAs) and pseudogenes that standard RefSeq might ignore. This is why GENCODE/Ensembl is preferred over UCSC/RefSeq for RNA-seq.

<br>

<div align="center">

| Version	Source	| Common Alias	| Chromosome Naming	| Annotation | Source	Notes |
| :--- | :--- | :--- | :--- | :--- |
| GRCh37 | Ensembl | —	1, 2, 3…MT | Ensembl 75 / GENCODE v19	| Older but stable |
| hg19 | UCSC | —	chr1, chr2…chrM | RefSeq / GENCODE v19 | Equivalent to GRCh37 |
| GRCh38 | Ensembl | —	1, 2, 3…MT | Ensembl ≥76 / GENCODE v40 | Current standard |
| hg38 | UCSC | —	chr1, chr2…chrM | RefSeq / GENCODE v40 | Equivalent to GRCh38 |

</div>

<br>

## Genome Indexing

If the aligner had to read the entire 3.2 billion base pairs of the human genome for every single read in the FASTQ files, the analysis would be incredibly slow and computationally infeasible. To overcome this, reference FASTA files are converted into a rapidly searchable database (an index) made up of short, overlapping subsequences known as **k-mers** (sequences of length *k*). This process is called **indexing**.

One important practical consideration is that each aligner generates a different index format, and they are not compatible. Since generating an index is computationally expensive, it is good practice to download a pre-built one (from [Illumina iGenomes](https://support.illumina.com/sequencing/sequencing_software/igenome.html), [AWS iGenomes](https://ewels.github.io/AWS-iGenomes/), or Ensembl/UCSC FTP) that can be used with the chosen aligner. 

## The Mapping Process

Aligners take short sequences (**seeds**) from the sequencing reads and try to find candidate alignment locations in the index. Once the aligner has "anchored" the read to a specific spot, it attempts to align the remainder of the read to the adjacent genomic coordinates, allowing for a predefined number of mismatches or gaps (indels). This is called the **seed-and-extend method** and it is essential because, if the aligner tried to find an exact 150-base match in the 3-billion-base human genome, it would almost always fail, due to the presence of point mutations and sequencing errors.

The aligner calculates an **alignment score** by rewarding matches and applying penalties for mismatches and indels. If a sequences maps to several locations, the aligner chooses the one with the highest alignment score but, if multiple locations have the same top score, the read is considered **multi-mapped**, which significantly lowers its Mapping Quality (MAPQ) score (see below).

<br>

<div align="center">
  
  <img src="../Figures/seed_and_extend.png" width="800">
  
  <br><br>
  
  <em>Read alignment with seed-and-extend approach. Adapted from Karpulevich et. al. BMC Bioinformatics (2024), used under CC BY 4.0 (https://creativecommons.org/licenses/by/4.0/deed.en)</em>
</div>

<br>

The result of this complex scoring and pairing process is recorded in a **SAM file**.

Sam files are human-readable text files, where each line is tab-delimited, showing:

- Read name
- Numerical flag. This is a compact way of storing multiple pieces of information about the read—such as whether it is paired, if it mapped to the forward or reverse strand, or if it failed to map entirely. To check what the number in this column means, the [Broad Institute SAM Flag Explainer](https://broadinstitute.github.io/picard/explain-flags.html) tool can be used.
- Chromosome
- Position
- **CIGAR string**. This is a compact code that describes the extend part of the mapping. For example, 150M means 150 bases matched perfectly, while 100M2D48M means 100 matches, a 2-base deletion, and 48 more matches.

<br>

<div align="center">

| Letter	| Meaning	| What it looks like |
|---------|---------|--------------------|
| M | Alignment match	| The bases align (can be a match or a mismatch) |
| I | Insertion	| The read has extra bases that the reference doesn't |
| D | Deletion | The reference has bases that the read is missing |
| S | Soft-clipping	| These bases are at the ends and didn't match, so the aligner ignored them |

</div>

<br>

**Note:** The presence of many S (Soft-clipping) in a SAM file usually means that the adapter trimming (with fastp or cutadapt) didn't work perfectly. The aligner is doing the work by hiding the adapters so the read can still map.

<br>

<div align="center">
  <img src="../Figures/SAM_file.png" width="800">
  <br>
  <em>Example SAM file showing common alignment outcomes</em>
</div>

<br>

Due to the immense size of sequencing data, SAM files are typically converted into **BAM** (Binary Alignment Map) files. BAM files are compressed, non-human-readable versions of SAM files that allow for much faster data processing and significantly reduced storage footprints. For most downstream analyses, BAM files must be **sorted** by genomic coordinate and **indexed** (creating a corresponding .bai file) to allow software to quickly access specific regions of the genome.

## Mapping Paired-end Reads

In a paired-end run, the aligner uses the physical properties of the DNA fragment in addition to the sequence to validate the mapping.
First, R1 and R2 are aligned independently. Once candidate locations are found, the aligner validates the pair based on three criteria:

 - **Chromosome:** both reads must map to the same chromosome.
 - **Orientation:** inward-facing for most Illumina libraries.
 - **Insert size:** the distance between R1 and R2 should match the library preparation.

If these rules are not fulfilled, the pair is flagged as **discordant** by the aligner, keeping both reads but lowering their MAPQ score. The presence of a high mapping rate, together with a low properly-paired rate signals a problem during library preparation, such as over-fragmentation or chimera generation in the PCR step. In standard pipelines, these are often filtered out as noise, but in cancer genomics, they are essential for identifying structural variants and chromosomal rearrangements.

## Selecting the correct fragment size

As mentioned in the [library preparation](./01_NGS_Foundations_&_Pre-processing/03_library_preparation.md) section of this repository, selecting a correct fragment size is critical for the alignment step, and the decision is made based on both the insert size and the read length. To illustrate this, a paired-end sequencing set to 150 bp read length will be used as an example:

- If the insert size is less than 300 bp, the R1 and R2 reads will overlap in the middle (**overlapping reads**). This provides double sequencing depth for the center of the fragment, which can be used to correct sequencing errors, but limits the overall genomic coverage.
- If inserts are over 400 bp, there will be an unsequenced gap in the middle, which the aligner will fill using paired-end logic. This reduces sequencing depth but allows for a higher coverage in exchange.
- Finally, if the insert is shorter than the read length (below 150 bp), both R1 and R2 will cover the full genomic sequence and read into the adapter on the other end. This provides very high sequencing depth but requires very heavy trimming to remove the adapter sequences.

<br>

<div align="center">
  <img src="../Figures/fragment_size.png" width="800">
  <br>
  <em>Fragment size selection logic.</em>
</div>

<br>

## Mapping Quality (MAPQ)

The MAPQ score is a logarithmic scale (Phred-scaled) that represents the probability that the alignment is wrong:

$$\text{MAPQ} = -10 \log_{10}(P_{err})$$

A MAPQ of 0 means that the sequence is multi-mapped: the aligner found two or more places in the genome where the read fits perfectly (e.g., in a repetitive element or a duplicated gene). Because the aligner can't be sure which one is right, it assigns a probability of zero that it chose the correct one.

Different factors determine the score:
-	**Uniqueness:** How much better is the best hit compared to the second best hit? If the best hit has 0 mismatches and the second best has 5 mismatches, the MAPQ will be high (~60).
-	**Base Quality:** If the bases that match the genome have low Phred scores (Q10 or Q15), the aligner is less confident in the match, and the MAPQ drops.
-	**Paired-End Information:** If R1 and R2 map to the same chromosome at the correct distance from each other, the MAPQ gets a "bonus" because the physical constraint of the DNA fragment confirms the location.

Importantly, different aligners use different MAPQ scores. This is critical when filtering BAMs with samtools in later steps of the analysis.

-	BWA-MEM: Max score is 60.
-	Bowtie2: Max score is 42.
-	STAR: Max score is 255 (Note: STAR uses 255 to mean "Uniquely Mapped").

## Mapping QC

After the aligner finishes, it generates a log file with some important statistics about the whole run, including:

- **% of mapped reads:** the closer to 100%, the better. Lower mapped reads can be a sign of wrong genome/annotation, contamination or poor library prep.
- **% of unique mapped reads and % of multi mapped reads:** most reads should be uniquely mapped, otherwise it could indicate over-amplification or low library complexity.
- **% of properly paired reads:** a low percentage here despite a high mapping rate often points to library prep issues (like presence of chimeras).



