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

## Coverage File Generation

Although coverage file generation is often treated as a straightforward format conversion step, it plays a critical role in shaping how signal is interpreted in downstream analyses. Errors introduced here will propagate into visualization, peak calling (in the case of SEACR, since it uses these files as input), and quantitative comparisons.

In a pipeline in which bedGraph files are generated from BAM files (normally with deepTools bamCoverage), and then converted to BigWig format for visualization in a genome browser with bedGraphtoBigWig, in the vast majority of cases, if a coverage track looks incorrect, **the problem originates in coverage generation**, not in the bedGraph file or the bedGraph → BigWig conversion step.

It is therefore easier to assess coverage file generation from the tracks that the BigWig files form in a genome browser.

### Inflated or compressed signal

Usually caused by a wrong normalization, this is one of the most common issues observed in a CUT&RUN protocol. Not using, or using an incorrect apike-in scaling factor results in signal differences reflecting sequencing depth rather than biology. Importantly, the scaling factor has to be applied to all samples, or the comparisons between them will likely not be valid. However, before applying it, it is important to look at the number of spike-in reads per sample: samples with very few spike-in reads will produce artificially inflated coverage, leading to noisy, over-amplified tracks, so their presence in the final quantification steps should be assessed.

### No signal in coverage tracks

- **Excessive removal of reads:** strict MAPQ thresholds or duplicate removal (using the *--ignoreDuplicates* parameter of bamcoverage) can eliminate real CUT&RUN signal.
- **Improper bin size selection:** while the default bin size used by bamcoverage is 50 bp, this can be modified with the *--binSize* parameter. A lower bin size will increase resolution, but will make the transformation more computationally demanding, and viceversa.
- **Use of read extension:** CUT&RUN does not require fragment extension; applying it with *--extendReads* artificially broadens peaks and reduces apparent enrichment.

### Unexpected enrichment patterns

If coverage profiles do not match the ones observed on the BAM files and/or they look biologically implausible:

- **Over-smoothing of signal:** Caused by large bin sizes or inappropriate parameters, leading to broad, ChIP-seq-like profiles instead of sharp CUT&RUN peaks.
- **Mis-centering of reads:** While centering reads (*--centerReads*) can improve apparent peak sharpness in transcription factor CUT&RUN experiments, it assumes relatively uniform fragment sizes. If applied to heterogeneous fragment populations (e.g., mixed nucleosomal and sub-nucleosomal fragments), it can distort peak shape and shift apparent binding sites. For this reason, centering is best restricted to analyses focusing on short fragments, and should be avoided in general or histone mark workflows.
- **Fragment size considerations:** Including all fragment sizes without filtering can obscure the distinction between TF binding (short fragments) and nucleosomal signal.

## Peak Calling & Interpretation

Even with high-quality alignments and clean fragment profiles, peak calling remains one of the most critical and error-prone steps in CUT&RUN analysis. The choice of algorithm, parameters, and controls can dramatically influence the final set of detected binding sites.
