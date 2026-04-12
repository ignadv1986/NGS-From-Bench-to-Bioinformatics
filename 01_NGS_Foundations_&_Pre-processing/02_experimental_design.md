# NGS Experimental Design

In addition to standard experimental design factors , such as conditions or number of replicates, NGS experiments require several parameters to be defined prior to library preparation and sequencing, including sequencing depth, replication strategy, read configuration, and total data output. Defining these correctly is essential to ensure efficient use of time and budget, and to avoid under- or over-sequencing.

## Sequencing Depth & Coverage

**Sequencing depth**, also known as read depth or depth of coverage, refers to the number of times a specific base (nucleotide) in the DNA is read during the sequencing process. In other words, it's the average number of times a given position in the genome is sequenced. A higher sequencing depth provides more confidence in the accuracy of the base calls at that position and helps to reduce sequencing errors and noise.
For example, if a specific nucleotide is sequenced 30 times, the sequencing depth at that position is 30x. This is the gold standard for whole genome sequencing (WGS) because, statistically, it ensures that >99.9% of the genome is covered by at least a few reads, minimizing the risk of missing a heterozygous variant.

Before the experiment is performed, the flow cell that will be used needs to be decided based on the aimed sequencing depth and the genome size:

$$\text{Total Data (Gb)} = \text{Genome Size (Gb)} \times \text{Desired Depth (} \times \text{)}$$

Based on this formula, a full sequencing of a human genome (around 3.2 Gb) at 30x sequencing depth, requires 96 Gb of raw data.
Choosing the right flow cell for the experiment can save money and time. High-output flow cells are more expensive and take more time to image, but they might not be ideal if the total data required by the experiment isn’t very high.

**Coverage** or **breadth of coverage** is closely related to sequencing depth but provides a broader perspective. Coverage is the proportion or percentage of a genome that has been sequenced at a certain depth. It gives an idea of how much of the entire genome has been effectively read and is usually expressed as a multiple of the genome's size, expressed as a percentage. For example, “95% coverage” means that 95% of the intended region has been sequenced at least once or a certain amount of times.

The higher the sequencing depth, the lower the possibility that some positions won't be sequenced or, in other words, the higher the coverage.

<div align="center">
  <img src="../Figures/sequencingdepth_vs_coverage.png" width="700">
  <br>
  <em>Representation of sequencing depth vs coverage</em>
</div>
