# Pipeline & Computational Troubleshooting (CUT&RUN)

Even when alignment rates and fragment size profiles look reasonable, CUT&RUN results can still be misleading during peak calling, normalization, and interpretation.

## Early Visualization in Genome Browsers

As mentioned in the [CUT&RUN analysis](../04_Epigenomics/02_CUT&RUN_analysis.md) section of this repository, after BAM file generation it is good practice to inspect the generated BAM files in a genome browser like IGV. A good CUT&RUN profile whould show sharp, discrete enrichment sites, with low background between them, and fragment size consistent with biology. Additionally, if binding sites for the protein of interest are known, enrichment at these sites should also be checked.

### Presence of signal across the whole genome

A high, uniform background in a CUT&RUN experiment can be caused by different factors:

- **Background contamination:** Even when not perfectly targeted, MNase tethered to antibody can still nick DNA in nearby accessible regions, especially in open chromatin regions such as promoters and nucleosome-depleted regions. In adition, ambient/carryover DNA can also contribute to an increase in the background signal.
- **Over-digestion:** The MNase activity needs to be tightly controlled during the experiment (time of activity, concentration, etc.), or it will diffuse off-target and start cleaving in a non-localized manner. While some of this background will be removed during the SPRI clean-up step, fragments of the selected size will remain, even if they didn´t originate from the correct cleaving sites.

### No visible enrichment

If no peaks are present when inspecting the BAM files in the genome browser, there can be both technical and biological reasons: - **Antibody choice:** Even when showing specificity in other assays, not all antibodies are suitable for CUT&RUN experiments. It is therefore recommendable to run a pilot experiment to check if the antibody is suited for this purpose, or to use an antibody previously documented to work for CUT&RUN. If the antibody fails to bind the protein of interest, the MNase won't be tethered to specific genomic regions and no enrichment will be visible.

- **Presence of protein of interest:** If the target protein is not expressed or is expressed at very low levels, the antibody won't have enouch targets to bind and no enrichment will be observed.
- **The protein of interest is not a chromatin binding factor:** If the aim of the CUT&RUN experiment is to determine if the protein of interest can bind to chromatin, then a negative result would also show no enrichment. However, the interpretation in this scenario is difficult, since the technical factors cited above cannot be ruled out.

### Peak shape does not match the expected outcome

Transcription factors generate sharp peaks, while histone binding factors lead to wider ones. The observed peak width therefore needs to match the nature of the targeted protein.

- **Wrong fragment selection:** As mentioned in the library preparation section, in CUT&RUN fragment selection varies depending on whether the protein of interest is a TF or a histone binding factor. If the fragment size selection is too permissive, peaks not corresponding to protein binding sites may appear.

### Peaks are present in non-expected regions

If known loci are empty but signal appears elsewhere, the issue is usually mislocalization of signal rather than complete failure of the experiment.

- **Wrong genome build or annotation mismatch:** If the genome assembly or annotation version does not match the expected reference, enrichment may appear “shifted” or assigned to incorrect genes/regions. This does not usually erase signal, but it makes expected loci appear empty while signal is observed elsewhere. This is especially relevant when comparing results to published datasets or known peak sets.
- **Artifacts in repetitive or low-complexity regions:** Some genomic regions inherently attract ambiguous alignments. If not properly accounted for, (usually in the blacklist region removal step of BAM clean-up) these regions can accumulate reads that resemble real peaks. In this case, signal may look strong and well-defined, but it will not correspond to known regulatory or binding sites. This often leads to apparent “new peaks” that are not reproducible or biologically meaningful.
