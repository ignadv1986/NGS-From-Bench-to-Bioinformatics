# NGS-From-Bench-to-Bioinformatics

A technical reference manual covering the full Next Generation Sequencing workflow, from sequencing chemistry and library preparation through to computational analysis and troubleshooting. Each step is covered in the context of how it affects the ones that follow, with particular emphasis on practical troubleshooting and experimental decision-making.

The troubleshooting sections are written from practical experience: each issue is traced back to its most likely origin in the experimental or computational workflow, with concrete diagnostic criteria and actionable recommendations.

This is a living document and will be updated over time.

---

## Contents

### 01 — NGS Foundations & Pre-processing

| File | Description |
| :--- | :--- |
| [01_sequencing_physics.md](./01_NGS_Foundations_%26_Pre-processing/01_sequencing_physics.md) | Sequencing by synthesis, bridge amplification, optical chemistry, patterned flow cells, and common run-level failure modes |
| [02_experimental_design.md](./01_NGS_Foundations_%26_Pre-processing/02_experimental_design.md) | Sequencing depth, coverage, read configuration, multiplexing strategy, and practical design trade-offs |
| [03_library_preparation.md](./01_NGS_Foundations_%26_Pre-processing/03_library_preparation.md) | End repair, adapter ligation, SPRI size selection, PCR amplification, quantification, and pooling |
| [04_sequencing_QC.md](./01_NGS_Foundations_%26_Pre-processing/04_sequencing_QC.md) | FastQC metrics, Phred scores, trimming strategy, and MultiQC |

### 02 — Mapping & Alignment

| File | Description |
| :--- | :--- |
| [01_mapping_principles_&_QC.md](./02_Mapping_%26_Alignment/01_mapping_principles_%26_QC.md) | Reference genomes, genome indexing, seed-and-extend alignment, SAM/BAM format, MAPQ, and mapping QC |
| [02_aligners.md](./02_Mapping_%26_Alignment/02_aligners.md) | STAR, HISAT2, BWA-MEM2, Bowtie2, Salmon, and kallisto — when and why to use each |
| [03_post-alignment_processing.md](./02_Mapping_%26_Alignment/03_post-alignment_processing.md) | BAM sorting, duplicate marking, blacklist filtering, spike-in normalization, and final indexing |

### 03 — Transcriptomics (RNA-seq)

| File | Description |
| :--- | :--- |
| [01_pre-processing.md](./03_Transcriptomics_(RNA-seq)/01_pre-processing.md) | RNA quality assessment (RIN), enrichment strategies, fragmentation, cDNA generation, strandedness, and UMIs |
| [02_analysis_workflow.md](./03_Transcriptomics_(RNA-seq)/02_analysis_workflow.md) | Read quantification with featureCounts, differential expression with DESeq2, LFC shrinkage, and functional enrichment |

### 04 — Epigenomics

| File | Description |
| :--- | :--- |
| [01_ChIP-seq_vs_CUT&RUN.md](./04_Epigenomics/01_ChIP-seq_vs_CUT%26RUN.md) | ChIP-seq principles, limitations, and a direct comparison with CUT&RUN |
| [02_CUT&RUN_sample_prep.md](./04_Epigenomics/02_CUT%26RUN_sample_prep.md) | pAG-MNase mechanism, fragment size biology, IgG control, SPRI strategy, PCR amplification, and quantification |
| [03_CUT&RUN_analysis.md](./04_Epigenomics/03_CUT%26RUN_analysis.md) | Spike-in normalization, coverage file generation, peak calling with SEACR and MACS3, replicate handling, and biological interpretation |
| [04_ATAC-seq_sample_prep.md](./04_Epigenomics/04_ATAC-seq_sample_prep.md) | Tn5 transposase mechanism, tagmentation, fragment size distribution, the 9 bp shift, and SPRI selection |
| [05_ATAC-seq_analysis.md](./04_Epigenomics/05_ATAC-seq_analysis.md) | Alignment strategy, mitochondrial read handling, TSS enrichment, coverage files, peak calling with MACS3, DiffBind, and differential accessibility |

### 05 — Troubleshooting

| File | Description |
| :--- | :--- |
| [01_fragment_size_distribution.md](./05_Troubleshooting/01_fragment_size_distribution.md) | Adapter dimers, broad distributions, high molecular weight contamination, and loss of expected fragment populations |
| [02_run_&_sequence_quality.md](./05_Troubleshooting/02_run_%26_sequence_quality.md) | Low Phred scores, GC content anomalies, adapter contamination, duplication patterns, and per-base N content |
| [03_alignment_&_data_quality.md](./05_Troubleshooting/03_alignment_%26_data_quality.md) | Low mapping rates, multi-mapping, and integrated interpretation of alignment metrics |
| [04_pipeline_&_computational(RNA-seq).md](./05_Troubleshooting/04_pipeline_%26_computational(RNA-seq).md) | featureCounts and Salmon quantification issues, DESeq2 model failures, and batch effect diagnosis |
| [05_pipeline_&_computational(CUT&RUN).md](./05_Troubleshooting/05_pipeline_%26_computational(CUT%26RUN).md) | Genome browser interpretation, coverage file generation, SEACR peak calling artifacts, and replicate inconsistency |
| [06_pipeline_&_computational(ATAC-seq).md](./05_Troubleshooting/06_pipeline_%26_computational(ATAC-seq).md) | TSS enrichment failures, coverage artifacts, MACS3 peak calling issues, featureCounts misconfiguration, and DESeq2 edge cases |

---

## License

This work is licensed under the [MIT License](./LICENSE). You are free to use, share, and adapt the material with attribution.

---

*For questions or collaborations, visit my [GitHub profile](https://github.com/ignadv1986).*
