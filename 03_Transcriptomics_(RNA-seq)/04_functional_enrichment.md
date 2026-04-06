# Functional Enrichment

The final stage of an RNA-seq pipeline is functional enrichment analysis. While DESeq2 provides a list of Differentially Expressed Genes (DEGs), enrichment analysis identifies which cellular pathways, biological processes, or molecular components are statistically overrepresented within that list.

## Categorization Frameworks

### GO (Gene Ontology)

The most widely used system, it categorizes genes into three structured domains:

- **Molecular function:** Activities at the molecular level (e.g., "ATP binding").
- **Biological process:** Larger "biological goals" (e.g., "DNA repair" or "signal transduction").
- **Cellular component:** Where the gene product is active (e.g., "Mitochondrial matrix").

### KEGG (Kyoto Encyclopedia of Genes and Genomes)

A database that maps genes to specific metabolic and signaling pathways, providing a "map" of how genes interact within a system

## Background Gene Selection

When doing functional enrichment, a critical step is choosing the right background genes list. Using the complete genome is not advisable, since not all genes are detected in most experiments and this can inflate the results. It is therefore more correct to use only the genes that were tested for differential expression in the first place.

## Software & Tools

The gold standard for functional enrichment analysis is the R/Bioconductor package **[clusterProfiler](https://bioconductor.org/packages/release/bioc/html/clusterProfiler.html)**. It allows for advanced visualization and GSEA (Gene Set Enrichment Analysis).
