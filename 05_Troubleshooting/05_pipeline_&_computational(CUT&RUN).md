# Pipeline & Computational Troubleshooting (CUT&RUN)

Even when alignment rates and fragment size profiles look reasonable, CUT&RUN results can still be misleading during peak calling, normalization, and interpretation.

## Early Visualization in Genome Browsers

As mentioned in the [CUT&RUN analysis](../04_Epigenomics/02_CUT&RUN_analysis.md) section of this repository, after BAM file generation it is good practice to inspect the generated BAM files in a genome browser like IGV. A good CUT&RUN profile whould show sharp, discrete enrichment sites, with low background between them, and fragment size consistent with biology. Additionally, if binding sites for the protein of interest are known, enrichment at these sites should also be checked.

**Note:** In practice, many analyses proceed directly to coverage files (BigWig) for visualization, as they are faster to load and easier to compare across samples. However, BigWig tracks represent a processed signal and can obscure read-level artifacts.
When peak calling or downstream analysis produces unexpected results, it is often necessary to return to the BAM files to diagnose the issue. Artifacts such as PCR duplicates, misaligned reads, or improper fragment handling can generate convincing but biologically meaningless signal that is not easily identifiable in coverage tracks.
Early inspection of BAM files therefore helps identify these issues before they propagate through peak calling and downstream analysis, reducing the risk of interpreting artifactual “ghost peaks” as real biological signal.

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

### Quick diagnostics guide

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Diffuse signal across genome (high background) | Over-digestion or background contamination | MNase conditions, fragment size profile, overall signal distribution |
| No visible enrichment anywhere | Antibody failure or missing protein | Antibody validation, expression of target protein |
| Very weak or barely detectable signal | Low protein abundance or inefficient targeting | Sequencing depth, antibody efficiency, experimental conditions |
| Peaks present but not sharp | Mixed fragment populations or poor digestion control | Fragment size selection, digestion time |
| Peak shape inconsistent with target (e.g., broad instead of sharp) | Incorrect fragment selection or wrong expectations | Fragment size distribution, protein type (TF vs histone) |
| Signal present in unexpected regions | Genome build mismatch or alignment artifacts | Reference genome version, annotation consistency |
| Strong peaks in repetitive regions | Poor handling of multi-mapping reads or missing blacklist filtering | Blacklist removal, alignment settings |

## Coverage File Generation

Although coverage file generation is often treated as a straightforward format conversion step, it plays a critical role in shaping how signal is interpreted in downstream analyses. Errors introduced here will propagate into visualization, peak calling (in the case of SEACR, since it uses these files as input), and quantitative comparisons.

In a pipeline in which bedGraph files are generated from BAM files (normally with deepTools bamCoverage), and then converted to BigWig format for visualization in a genome browser with bedGraphtoBigWig, in the vast majority of cases, if a coverage track looks incorrect, **the problem originates in coverage generation**, not in the bedGraph file or the bedGraph → BigWig conversion step.

It is therefore easier to assess coverage file generation from the tracks that the BigWig files form in a genome browser.

### Inflated or compressed signal

Usually caused by a wrong normalization, this is one of the most common issues observed in a CUT&RUN protocol. Not using, or using an incorrect spike-in scaling factor results in signal differences reflecting sequencing depth rather than biology. Importantly, the scaling factor has to be applied to all samples, or the comparisons between them will likely not be valid. However, before applying it, it is important to look at the number of spike-in reads per sample: samples with very few spike-in reads will produce artificially inflated coverage, leading to noisy, over-amplified tracks, so their presence in the final quantification steps should be assessed.

### No signal in coverage tracks

- **Excessive removal of reads:** strict MAPQ thresholds or duplicate removal (using the `--ignoreDuplicates` parameter of bamcoverage) can eliminate real CUT&RUN signal.
- **Improper bin size selection:** while the default bin size used by bamcoverage is 50 bp, this can be modified with the `--binSize` parameter. A lower bin size will increase resolution, but will make the transformation more computationally demanding, and viceversa.
- **Use of read extension:** CUT&RUN does not require fragment extension; applying it with *--extendReads* artificially broadens peaks and reduces apparent enrichment.

### Unexpected enrichment patterns

If coverage profiles do not match the ones observed on the BAM files and/or they look biologically implausible:

- **Over-smoothing of signal:** Caused by large bin sizes or inappropriate parameters, leading to broad, ChIP-seq-like profiles instead of sharp CUT&RUN peaks.
- **Mis-centering of reads:** While centering reads (`--centerReads`) can improve apparent peak sharpness in transcription factor CUT&RUN experiments, it assumes relatively uniform fragment sizes. If applied to heterogeneous fragment populations (e.g., mixed nucleosomal and sub-nucleosomal fragments), it can distort peak shape and shift apparent binding sites. For this reason, centering is best restricted to analyses focusing on short fragments, and should be avoided in general or histone mark workflows.
- **Fragment size considerations:** Including all fragment sizes without filtering can obscure the distinction between TF binding (short fragments) and nucleosomal signal.

### Quick diagnostics guide

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| One sample shows globally higher signal than others | Incorrect or inconsistent spike-in normalization | Scaling factor, spike-in read counts |
| Noisy / over-amplified signal across genome | Inflation due to low spike-in counts | Spike-in counts per sample |
| Signal appears overly smooth or broad | Bin size too large / over-smoothing | `--binSize` |
| Peaks appear artificially wide | Inappropriate read extension | `--extendReads` |
| Weak or missing signal compared to BAM | Over-filtering of reads | MAPQ filtering, duplicate removal |
| Coverage does not match BAM visualization | Incorrect coverage parameters | Consistency of `bamCoverage` settings |
| Differences between replicates introduced at this step | Inconsistent normalization or parameters | Scaling factors, identical processing |

## Peak Calling & Interpretation

Even with well-behaved coverage profiles, peak calling remains one of the most critical and error-prone steps in CUT&RUN analysis. This is particularly relevant when using SEACR, which relies on a global thresholding approach rather than complex statistical modeling. Because of this, SEACR has very limited parameterization (primarily *relaxed* vs *stringent* modes), and its behavior is largely dictated by the structure and distribution of the input signal, rather than user-defined thresholds.

Importantly, SEACR will always return a set of peaks. The presence of peaks alone does not guarantee biological relevance, and must always be interpreted in the context of signal quality and reproducibility.

### Excessive number of peaks

If peak calling returns a very large number of peaks despite reasonable-looking coverage tracks:

- **Overly permissive thresholds:** Running SEACR in *relaxed* mode can classify background fluctuations as peaks.
- **Sensitivity to low-level noise:** Even small variations in low-background CUT&RUN data can be interpreted as enrichment by threshold-based methods.

### Low number of detected peaks

If SEACR returns very few peaks, even when enrichment is visible in coverage tracks:

- **Too stringent filtering of peaks:** While the *stringent* mode helps preventing unspecific peaks detection, it also filters out real, low-signal peaks, so it should be used carefully.
- **Low contrast between peaks and background:** If enrichment is shallow relative to background, SEACR may fail to distinguish peaks.

### Peaks detected in control (IgG) samples

- **Structured background signal:** Open chromatin regions can produce consistent low-level signal, even in controls.
- **Threshold sensitivity:** In *relaxed* mode, these regions may exceed the detection threshold.

This does not necessarily indicate a failure of the experiment, but highlights the importance of comparing against controls during interpretation.

### Quick diagnostics guide

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Excessive number of peaks | Permissive detection in relaxed mode | SEACR mode, background signal |
| Very few peaks despite visible enrichment | Stringent mode filtering out low-signal peaks | SEACR mode, signal-to-noise ratio |
| Peaks detected in IgG control | Structured background signal | Control track, SEACR mode |
| Peaks in unexpected genomic regions | Alignment artifacts or annotation mismatch | Genome build, blacklist filtering |
| Peaks present but weak or not convincing | Marginal signal above threshold | Coverage tracks vs peak locations |

### Inconsistent peak detection across replicates

Even when coverage profiles appear similar, peak calling results may differ substantially between replicates when using SEACR.

- **Peaks present in merged data but absent in individual replicates:** Weak enrichment signals that are consistently present across replicates may not exceed the detection threshold individually, but can accumulate in merged data and be called as peaks.
- **Poor overlap between replicate peak sets:** Small differences in signal intensity can cause regions to fall above the global threshold in one replicate but below it in another, leading to inconsistent peak detection.
- **Variable peak boundaries:** Because SEACR defines peaks based on a global threshold, slight differences in signal shape can shift peak edges between replicates.

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Peaks present in merged data but absent in individual replicates | Signal accumulation across replicates | Compare merged vs individual peak sets |
| Poor overlap between replicate peak sets | Threshold sensitivity to small signal differences | Replicate consistency, signal strength |
| Inconsistent peak boundaries across replicates | Threshold-based peak definition | Peak coordinates across replicates |
| Many peaks lost after intersection | Weak or borderline enrichment | Signal strength, replicate quality |
| Large number of non-reproducible peaks | Noise-driven peak detection | Intersection criteria (e.g., 2/3 replicates) |
| Final peak set differs from individual replicates | Merging artifacts or inconsistent signal | Merged vs intersected peak comparison |

