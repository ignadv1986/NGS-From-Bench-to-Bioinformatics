# Sequencing Data QC

## Sequence Depth VS Coverage

**Sequencing depth**, also known as read depth or depth of coverage, refers to the number of times a specific base (nucleotide) in the DNA is read during the sequencing process. In other words, it’s the average number of times a given position in the genome is sequenced. A higher sequencing depth provides more confidence in the accuracy of the base calls at that position and helps to reduce sequencing errors and noise.
For example, if a specific nucleotide is sequenced 30 times, the sequencing depth at that position is 30x.

**Coverage** is closely related to sequencing depth but provides a broader perspective. Coverage is the proportion or percentage of a genome that has been sequenced at a certain depth. It gives an idea of how much of the entire genome has been effectively read and is usually expressed as a multiple of the genome’s size, expressed as a percentage. For example, “95% coverage” means that 95% of the intended region has been sequenced at least once or a certain amount of times.

One important term is **uniformity of coverage**: you can have 30x average depth, but if half the genome is at 60x and the other half is at 0x, your experiment is a failure. For WGS, we want flat coverage. For ATAC-seq or CUT&RUN, we actually want bumpy coverage (peaks where the protein binds and valleys everywhere else). 

