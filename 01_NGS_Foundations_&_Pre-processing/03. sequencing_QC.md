# Sequencing Data QC

## Sequence Depth VS Coverage

**Sequencing depth**, also known as read depth or depth of coverage, refers to the number of times a specific base (nucleotide) in the DNA is read during the sequencing process. In other words, it's the average number of times a given position in the genome is sequenced. A higher sequencing depth provides more confidence in the accuracy of the base calls at that position and helps to reduce sequencing errors and noise.
For example, if a specific nucleotide is sequenced 30 times, the sequencing depth at that position is 30x. This is the gold standard for whole genome sequencing (WGS) because, statistically, it ensures that >99.9% of the genome is covered by at least a few reads, minimizing the risk of missing a heterozygous variant.

Before the experiment is performed, the flow cell that will be used needs to be decided based on the aimed sequencing depth and the genome size:

**Total data (Gb) = Genome size (Gb) x Desired sequencing depth (x)**

For example, a full sequencing of a human genome (around 3.2 Gb) at 30x sequencing depth, requires 96 Gb of raw data.
Choosing the right flow cell for the experiment can save money and time. High-output flow cells are more expensive and take more time to image, but they might not be ideal if the total data required by the experiment isn’t very high.

Once the run is over, the actual sequencing depth achieved can be calculated through the following formula:

**Sequencing depth = Total number of reads x fragment length (bp) / Total genome size (bp)**

If a bacterial genome of 5 Mb is sequenced, and we obtain 2000000 reads of 150 bp, then the sequencing depth would be 60x

**Coverage** or **breadth of coverage** is closely related to sequencing depth but provides a broader perspective. Coverage is the proportion or percentage of a genome that has been sequenced at a certain depth. It gives an idea of how much of the entire genome has been effectively read and is usually expressed as a multiple of the genome's size, expressed as a percentage. For example, “95% coverage” means that 95% of the intended region has been sequenced at least once or a certain amount of times. It is calculated by bioinformatics tools (bedtools or samtools) after the mapping step, with the following formula:

**Coverage (%) = (Number of bases with at least one read / Total genome size) x 100**

The higher the sequencing depth, the lower the possibility that some positions won’t be sequenced or, in other words, the higher the coverage
| Coverage | Sequencing Depth |
|----|----|
| 95%	| ~3x to 5x average |
| 99%	| ~8x to 10x average |
| 99.99% | ~20x to 30x average |

One important term is **uniformity of coverage**. This is a measure of the variability of coverage across the genome. While the overall coverage might be the desired one, this is an average across all positions, so some bases might not be reaching it while others have an inflated coverage, skewing the %. This can be problematic, especially or WGS, where we want flat coverage. If a position shows a much lower coverage, then the detection of mutations with high confidence won’t be possible. For ATAC-seq or CUT&RUN, we actually want bumpy coverage (peaks where the protein binds and valleys everywhere else).

The most common cause of low uniformity of coverage is **PCR bias**. Some regions are easier to amplify by the polymerase, including those with lower GC content, leading to an overrepresentation of these fragments.

The uniformity of coverage is usually calculated by tools like Picard or Mosdepth with the following formula:

**Coefficient of Variation (CV) = Standard Deviation of Depth / Average Sequencing Depth**. The lower this metric (0.1-0.2), the more uniform the data.

## Sequencing QC

- **Per Base Sequence Quality**

It shows the average **Phred score** for each position in the reads. The Phred score measures how confident the sequencer is that the detected base is the correct one, so the higher the score, the better the quality. The Pred score is calculated as **Q = -10log<sub>10</sub>(P)**, where P is the probability of an incorrect base call. Q30, the usually aimed for Phred score, translates to a 1 in 1000 error rate, or a 99.9% accuracy. Scores above 28 are categorized as good quality, and the score should remain uniform throughout the whole sequence. A drop at the end is normal in Illumina sequencers and it's usually solved during the trimming process.

- **Per Sequence Quality Score**

Average Phred score of the bases constituting each read sequence. It shows as a histogram with ideally a unique peak at high Phred scores.

- **Per Base Sequence Content**

Shows the proportion of each nucleotide for each position in our reads. This proportion should be roughly the same for all nucleotides. We should therefore see straight lines very close to each other for all nucleotides. 
The presence of wavy lines at the beginning of the reads is usually caused by adapters or random hexamer primer bias in the case of RNA-seq. In ATAC-seq, the Tn5 Transposase has a specific "insertion bias". 
Differences in G-C vs A-T might have a biological meaning, so worth investigating, while sudden peaks are usually a red flag.

- **Per Sequence GC Content**

This indicates the percentage of AT-GC bases for all reads, represented as a histogram.
The GC distribution should form a smooth, bell-shaped curve (approximately normal), matching the GC content of the organism of interest (around 50% in humans).
If we see two peaks instead of one, that might be a sign of contamination with DNA from a different species. 

- **Per Base N Content**

N is referred by the sequencer as bases that could not be properly identified. Obviously, this number should be close to 0 for all reads.
As little as an increase to 1% in any position is already a bad sign.
A raise towards the end of the ends might be normal and depending on the size it might be worth trimming.

- **Sequence Length Distribution**
 
Shows the distribution of the reads' lengths. All reads should be the same size in a successful sequencing, unless different adapters or trimming attributes were applied.

- **Sequence Duplication Levels**

Shows the percentage of repeated reads, and how many times they are present. The title of the graph tells us what percentage of reads we would be left with if we removed all duplicated reads (so the percentage of unique reads). 
This really changes depending on the type of samples we have. In RNA-seq, a high number of duplicated reads is expected, since many reads will map to abundant transcripts, while a high number of duplicates in WGS usually means that the library was overamplified. 

- **Overrepresented sequences**

A sequence is reported if it makes up >0.1% of total reads. This can happen for adapters or primers, which would require further processing of the samples.

- **Adapter Content**

Especifically detects the presence of adapters. It should be 0, or otherwise trimming is required.

## Trimming

As mentioned above, some sequences, such as **adapters, low quality bases, and poly-N tails** can affect downstream mapping and therefore need to be removed. This process is called trimming, and it can be achieved with different tools. The usual strategy is to run and initial round of fastQC to check the raw state of the run, followed by fastp to remove adapters and other non-desired elements and adding a QC report. It is good practice to run the cleaned data again through fastQC to see how the quality has improved.
When more customization is needed, **cutadapt** is the preferred option, since it supports more complex trimming rules. This would be the case for small RNA-seq experiments and/or when variable length adapters were used. 
Usually, trimming adapters is enough; the aligner (like BWA or STAR, see below) can handle a few low-quality bases at the ends.


