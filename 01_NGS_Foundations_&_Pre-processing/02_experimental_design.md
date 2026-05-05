# NGS Experimental Design

In addition to standard experimental design factors, such as experimental conditions and number of replicates, NGS experiments require several parameters to be defined prior to library preparation and sequencing, including sequencing depth, replication strategy, read configuration, and total data output. Defining these correctly is essential to ensure efficient use of time and budget, and to avoid under- or over-sequencing.

## Number of Samples and Replicates

The number of samples and replicates is primarily determined by the biological question and experimental design. However, these factors have a direct impact on sequencing requirements and must be considered when planning data output and flow cell usage.

Biological replicates are essential to capture biological variability and ensure robust downstream analysis. In most cases, increasing the number of biological replicates provides greater statistical power than increasing sequencing depth beyond recommended levels.

From a sequencing perspective, there is a trade-off between the number of samples/replicates and the depth achieved per sample. Given a fixed sequencing capacity, increasing the number of replicates reduces the number of reads allocated to each sample, while unnecessarily increasing sequencing capacity leads to higher experimental costs. Therefore, when designing the experiment, deciding which is the limiting factor is critical.

As a general principle, increasing the number of biological replicates provides greater statistical power than increasing sequencing depth beyond recommended levels. This is because sequencing depth addresses technical noise (the probability that a given base or transcript is detected) while replicates address biological variability, which is almost always the dominant source of uncertainty in a well-prepared library.

The practical implication is that once the recommended depth for a given application has been reached, additional reads yield diminishing returns. For example, in standard RNA-seq, this threshold is typically around 20–30 million reads per sample for well-annotated genomes. Sequencing the same sample at 60 million reads will not double the number of detected differentially expressed gene, but adding a third biological replicate to a two-replicate design often will, by giving DESeq2 the variance estimates it needs to distinguish signal from noise.

The trade-off can be framed as follows:

<br>

<div align="center">
  
| Situation | Recommendation | Reasoning |
| :--- | :--- | :--- |
| At or above recommended depth, fewer than 3 replicates | Add replicates | Statistical power is replicate-limited, not depth-limited |
| Below recommended depth, sufficient replicates | Increase depth | Coverage gaps will compromise detection regardless of replication |
| Highly variable biological system (e.g. primary cells, patient samples) | Prioritize replicates | Biological variance dominates; depth cannot compensate |
| Rare transcript detection or low-input samples | Increase depth | Low-abundance targets require more reads to clear the detection threshold |
| Fixed budget, forced choice | ≥3 replicates at minimum depth over 2 replicates at high depth | A statistically untestable experiment produces no publishable result |
</div>

<br>

## Sequencing Depth & Coverage

**Sequencing depth**, also known as read depth or depth of coverage, refers to the number of times a specific base (nucleotide) in the DNA is read during the sequencing process. In other words, it's the average number of times a given position in the genome is sequenced. A higher sequencing depth provides more confidence in the accuracy of the base calls at that position and helps to reduce sequencing errors and noise.
For example, if a specific nucleotide is sequenced 30 times, the sequencing depth at that position is 30x. This is the gold standard for whole genome sequencing (WGS) because, statistically, it ensures that >99.9% of the genome is covered by at least a few reads, minimizing the risk of missing a heterozygous variant.

Before the experiment is performed, the flow cell that will be used needs to be decided based on the aimed sequencing depth and the genome size:

$$\text{Total Data (Gb)} = \text{Genome Size (Gb)} \times \text{Desired Depth (} \times \text{)}$$

Based on this formula, a full sequencing of a human genome (around 3.2 Gb) at 30x sequencing depth, requires 96 Gb of raw data.

**Coverage** or **breadth of coverage** is closely related to sequencing depth but provides a broader perspective. Coverage is the proportion or percentage of a genome that has been sequenced at a certain depth. It gives an idea of how much of the entire genome has been effectively read and is usually expressed as a multiple of the genome's size, expressed as a percentage. For example, “95% coverage” means that 95% of the intended region has been sequenced at least once or a certain amount of times.

The higher the sequencing depth, the lower the possibility that some positions won't be sequenced or, in other words, the higher the coverage.

<div align="center">
  <img src="../Figures/sequencingdepth_vs_coverage.png" width="700">
  <br>
  <em>Representation of sequencing depth vs coverage</em>
</div>

<br>

## Library Pooling and Demultiplexing

To optimize sequencing capacity, multiple samples are often pooled and sequenced together using unique index sequences, a process known as multiplexing (see the [library preparation](./03_library_preparation.md) section of this repository).

When multiple samples need to be sequenced together, the total data output must be distributed accurately across the pool. Given a fixed sequencing capacity, the number of reads allocated per sample decreases as more samples are added. Careful planning is therefore required to ensure that each sample reaches its minimum required depth.

The starting point is always the per-sample data requirement:

$$\text{Total Data (Gb)} = \text{Genome Size (Gb)} \times \text{Desired Depth (} \times \text{)}$$

Following the example cited above, for a human WGS experiment at 30x, this is approximately 96 Gb per sample. If 24 samples need to be sequenced at this depth, the total data requirement is approximately 2.3 Tb. This immediately defines the minimum sequencing capacity needed, and whether that fits within a single run or requires multiple runs is a question for the sequencing facility — but the biological and financial implications of the number are clear regardless of instrument.

From a pooling perspective, two failure modes need to be avoided:

- **Under-filling a high-capacity run** wastes sequencing capacity and increases the cost per sample without any biological benefit. If only 6 of a possible 24 samples are ready, it is usually more cost-effective to either wait for the remaining samples or use a lower-output run configuration.
- **Over-pooling samples with unequal molarity** leads to uneven read distribution, where some samples are over-sequenced and others fall below the required depth. This is why the normalization calculation before pooling is critical

## Read Configuration

Sequencing experiments must also define the read configuration, including read length and whether sequencing is performed in single-end or paired-end mode.

While these topics are covered elsewhere in this repository, it is important to note here that:

- Longer reads improve alignment accuracy and detection of structural features, while shorter reads reduce cost and increase throughput.
- Paired-end sequencing provides additional information by sequencing both ends of each fragment, improving mapping accuracy and variant detection, but requires more sequencing capacity, increasing cost.

The optimal configuration depends on the application and must be balanced against cost and data requirements.

## Flow Cell Choice

Choosing the appropriate flow cell is critical for cost efficiency. Flow cells differ in total data output, and the optimal choice depends on the number of samples, biological replicates, and the required sequencing depth. Smaller flow cells are more suitable for low sample numbers or pilot experiments, as they minimize unused capacity. In contrast, high-output flow cells reduce cost per base but are only cost-effective when fully utilized. Underfilling a high-capacity flow cell can significantly increase the cost per sample, making careful planning essential.

Flow cell selection is instrument-specific and should be determined in consultation with the sequencing facility or Illumina documentation, based on the total data output required.

<br>

<div align="center">
  
| Project Scale | Data Requirement | Recommended Strategy | Cost Consideration |
|--------------|------------------|----------------------|--------------------|
| Small (pilot, few samples) | Low (≤100 Gb) | Use low-output runs | Minimizes wasted capacity |
| Medium (tens of samples) | Moderate (100–1000 Gb) | Use mid-output runs or multiplex samples | Balance between flexibility and cost |
| Large (cohort studies, WGS) | High (≥1 Tb) | Use high-output runs | Lowest cost per Gb if fully utilized |

</div>

