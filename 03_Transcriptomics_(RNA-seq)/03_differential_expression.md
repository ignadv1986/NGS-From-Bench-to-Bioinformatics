## Differential Expression: DESeq2

For the identification of differentially expressed genes, the R/Bioconductor package DESeq2 is the industry standard. It works on a **DESeq2 object**, constructed from a count matrix (usually provided by featureCounts) with the number of reads, with the genes as rows and the samples as columns, and a metadata matrix containing sample information (replicate, condition…).

Example of a count matrix:

| Gene | Sample1 | Sample2 | Sample3 | Sample4 | Sample5 | Sample6 |
|------|---------|----------|---------|---------|----------|---------|
| GeneA | 123 | 98 | 112 | 304 | 350 | 335 |
| GeneB | 7 | 15 | 4 | 6 | 7 | 12 |
| GeneC | 300 | 250 | 289 | 150 | 123 | 98 |

Example of a metadata matrix 

| Sample | Condition | Replicate |
|--------|-----------|-----------|
| Sample1 | Control | 1 |
| Sample2 | Control | 2 |
| Sample3 | Control | 3 |
| Sample4 | Treated | 1 |
| Sample5 | Treated | 2 |
| Sample6 | Treated | 3 |
