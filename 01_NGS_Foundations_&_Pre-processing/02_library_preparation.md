# Library Preparation

## Library preparation steps

Understanding the chemistry of reversible terminators explains how a single molecule is read. However, to achieve massively parallel sequencing, we must first transform raw biological samples (DNA or RNA) into a format the sequencer can recognize, capture, and amplify. This brings us to **library preparation**.

In the first step of library preparation, the DNA (or cDNA generated from retro-transcribed RNA) is cut in specified sizes through sonication or enzymatic digestion.

Next, small pieces of DNA, known as **adapters**, are added to the DNA. These fragments, specific for the different sequencers, contain the P5/P7 (flow cell binding, see below), priming sites and index/barcodes, and bind to the end of of the library´s DNA. This is the simplest workflow, mandatory for PCR-free protocols (e.g protocols that yield enough DNA that they do not require a PCR amplification step), but also more expensive and requires a different adapter for every single sample (since the index is already on the adapter).

However, the most common high-throughput method involves the use of truncated (stubby) adapters, known as **TA-ligation with indexed PCR** or **universal adapter ligation**. In this protocol, adapters contain only the initial priming sites, and the library is "completed" during a subsequent indexed PCR step, where primers add the barcodes and the P5/P7 flow-cell binding sequences. If this PCR fails, then the sequencing will also fail, since the fragments won´t contain the P5/P7 needed for the binding.

In order for the adapters to bind to the DNA, a previous step of **end repair** is required, where all fragment ends are converted to blunt, by removing the 3´overhangs and filling in the 5´, and subsequently 5´phosphorylated. Finally, a single A is added to the 3´end of each fragment (**“A-tailling”**). Adapters, which contain a a complementary single 3′-T overhang, can now be added and bound to the fragments by a DNA ligase. T-A ligation is much more efficient than blunt-end, prevents the fragments from binding each other, which would create chimeras, and reduces (although doesn´t eliminate) adapter dimers. 
As we will see later, ATAC-seq uses tagmentation, so there´s no end-repair, A-tailing, or ligation. 

A crucial step in library prep is the **selection of fragments of the right size**. This is mainly done through the use of Solid Phase Reversible Immobilization (SPRI) beads, like AMPure XP. The “bead mix” consists of three things:

-	Paramagnetic beads: iron cores coated in carboxyl groups (negative charge)
-	Polyethylene glycol (PEG)
-	NaCl
  
The process relies on charge-shielding rather than direct molecular bridging. High concentrations of NaCl provide a dense population of Na+ counter-ions that screen the electrostatic repulsion between the negatively charged DNA backbone and the carboxylated bead surface. This allows the crowding agent (PEG) to thermodynamically drive the DNA out of the aqueous phase, forcing it to collapse onto and entangle with the bead surface. This interaction is purely concentration-dependent; upon the addition of a low-salt elution buffer, the shielding is lost, and the restored electrostatic repulsion facilitates the release of the DNA.
Large DNA molecules dehydrate more easily, so they bind to the beads even at lower PEG concentrations, while small molecules are more soluble and therefore require higher PEG concentrations to bind the beads. Using different bead volume to sample volume ratios, different DNA sizes can be selected. A lower ratio (0.5x) captures only very large fragments, while a higher ratio (1.8x), captures almost everything.
To make sure that only the fragments of the right size are captured, a double-sided selection is normally used. First, a “right-side cut”, where a really low ratio (0.5x) is used, is performed. In this scenario, large pieces of DNA bind to the beads, and the rest stays on the liquid, so the beads can be discarded. Then, more beads are added to bring the total ratio up to, say, 0.8x (“left-side cut”), so that the rest of the DNA, except for small pieces like primer dimers stick to the beads. These are kept, washed with ethanol 80% to remove contaminants, and finally eluted with water or a low-salt buffer. 80% ethanol is used because it’s strong enough to keep the DNA precipitated on the bead, but contains enough water to dissolve the salts and PEG so they can be washed away.

When performing the size selection step, two measurements need to be considered:

- The **fragment size** is the total length of the DNA fragment (genomic DNA + adapters)
- The **insert size** refers only to the genomic DNA between the two adapters

The relationship between the fragment size and the **read length**, a setting that can be selected in the sequencer, determines how the data will look to the aligner. A detailed breakdown of fragment size choice can be found in the [mapping](02. Mapping & Alignment/mapping) section of this repository.

Lastly, in some protocols the library is amplified with a PCR step, to increase the concentration of DNA (see next section). In **PCR-free NGS protocols**, like whole genome-sequencing (WGS), we start with a much higher initial concentration of DNA (like 1 µg), so there is already enough material to be sequenced after adapter ligation. This has some advantages: PCR polymerases naturally "dislike" areas with high GC content (promoters) or high AT content. PCR-free sequencing provides the most even coverage across the entire genome because you remove the "enzyme preference" entirely. Additionally, PCR can sometimes introduce small insertions or deletions (stutter). PCR-free is superior for clinical variant calling. On the downside, PCR-free NGS protocols contain molecules with both, only one, or no adapters. This is because such adapters are used for library amplification in libraries that have a PCR step, so virtually all fragments will contain both adapters. However, in PCR-free protocols, the adapters are not used for amplification and there is no way to guarantee that all fragments will bind to both adapters.

## Library amplification by PCR

The goal of library PCR is to add the remaining adapter sequences (if using indexed primers) and to amplify the library to a measurable concentration (typically 2–10 nM for loading). When deciding on the number of PCR cycles, two scenarios need to be avoided:

-	**Under-amplification:** Leads to a library concentration below the detection limit of the Qubit or TapeStation (<0.5ng/µl), making accurate loading impossible.
-	**Over-amplification:** Leads to PCR duplicates (reducing unique data) and heteroduplexes (the "bubble product").

| Input DNA | Typical cycle range |
|-----------|---------------------|
| 1 µg (WGS) | 0-4 cycles |
| 50–100 ng (standard) | 5-8 cycles |
| 1–10 ng (low input) | 10-15 cycles |
| <1 ng (single-cell, ultra-low) | 18+ cycles |

**PCR duplicates:** Because some DNA fragments can be amplified more efficiently than others (medium GC content, no hairpins, etc.), when we amplify for too long we risk getting an overrepresentation of these fragments in the final sample. These can take more space in the flow cell, preventing detection of other fragments that, even though were present in the original sample, were not amplified as much. This is often called a reduction in library complexity or diversity.

**The Bubble Product (heteroduplex):** In the final stages of the library PCR, primers get depleted (they run out) and there is an overabundance of DNA fragments. Instead of a primer binding to a template, two full-length library fragments denature (separate) and then accidentally anneal (re-bind) to each other. Since the adapters are identical for all fragments, they zip up perfectly. However, the genomic inserts (the middle part) are different. They are not complementary. The result is a DNA molecule that is double-stranded at the ends (the adapters), but single-stranded in the middle, forming a bubble (heteroduplex). These molecules are less dynamic and migrate slower in an electrophoresis, so they give rise to a high molecular weight peak . However, the sequencer denatures the dsDNA, so this will have no consequences on the sequencing itself. This is a problem of overamplification, so reducing the amount of PCR cycles to reduce reactive use is recommended.

**Thing to consider when designing the library PCR:**

-	**Use high-fidelity polymerases** (KAPA HiFi, Q5 (NEB), or Phusion): Less error-rates and GC bias than taq polymerases, and proof-reading activity (exonuclease 3´-> 5´).
-	**Hot-Start technology:** Prevents non-specific amplification at room temperature before the thermocycler starts. This is crucial for reducing Primer Dimers.
-	**Final concentration:** A well-designed PCR should aim for a final library concentration of >10 nM. This provides enough material for multiple sequencing runs and long-term storage.

## Library Quality Control

To ensure library integrity, fragment size and concentration are assessed before sequencing. Fragment size distribution is analyzed with instruments such as the **BioAnalyzer** or the **TapeStation** to ensure that DNA is in the expected size range. For library quantification, while Nanodrop can be used as a quick first check, **Qubit**, a fluorescence based method where only dsDNA is quantified, as opposed to Nanodrop, which detects all nucleic acids, is preferred. This is the case for libraries that have undergone an amplification step. However, in PCR-free protocols, as mentioned before, many fragments do not have both adapters, and therefore they won't be sequenced. Qubit will nevertheless detect these fragments, leading to an inflation in the amount of sequencing-ready DNA present in the sample. Therefore, in these protocols, the concentration must be calculated by **qPCR-based quantification** with KAPA/NEB Library Quant, which uses primers that bind to the P5/P7 adapters.

- **Fragment Size**

Both Tapestation and Bioanalyzer are microfluidic electrophoresis instruments, while the fragment analyzer uses capillary electrophoresis. For all of them, the principle is the same: samples are loaded together with a fluorescent dye and a molecular size marker and fragments are subjected to an electric field so that they separate by size. The instruments detects the fluorescent signal vs time and transforms it into bp using the provided ladder.
Tapestation takes more samples than Bioanalyzer, and the sample preparation is easier, so it´s usually the preferred option. However, the fragment analyzer has higher throughput and precision than the other two, allowing greater resolution and distinction between small framents and primer dimers or other artifacts.
The result from fragment size analysis is presented as both an **electropherogram** and a **virtual gel**. 

Ideally, the tapestation returns a main peak with the desired fragment size, that varies in width depending on the quality of the library. A smaller peak (around 120-140 bp for full adapters, or 60-80 for truncated ones) is usually seen corresponding to adapter/primer dimers. Because full primers are longer, then the likelihood of forming dimers is higher. This can be problematic for several reasons:

-	Smaller molecules physically diffuse to the flow cell surface and "capture" a grafting oligo much faster than long library fragments (300-500 bp).
-	Because they are short, the "bridge" is easier to form, and they amplify more efficiently during cluster generation **(cluster side bias)**.

Even if the library has only 5% dimers by mass (ng), they can take up 50% or more of the "clustering occupancy." Libraries with a percentage below 5% of primer dimers are usually acceptable for sequencing.
It is advisable to use a hot-start polymerase (see above), and sometimes to increase the annealing Temperature 2°C with problematic libraries to increase stringency. Another important metric is the primer concentration added into the mix. The final concentration of each primer should be between 0.2-0-5 µM, but it's essential to take into account the primer-to-template ratio. If you have too much primer relative to very few DNA fragments, the primers are more likely to find each other (dimerization) than they are to find a rare DNA template, but we still need to make sure that to ensure the reaction doesn't "starve" before a detectable library concentration is reached. 
The solution, if all measures were taken and there is still over 5% of primer dimers, is a further 0.6x or 0.8x AMPure bead cleanup to "right-size" the library. A ratio (e.g., 40µL beads to 50µL DNA) is the "sweet spot" for removing 120 bp dimers while keeping 300 bp+ libraries.

When the DNA/RNA has been degraded during the process, the electropherogram shows a very wide peak, that sometimes fuses with the primer dimer one, and the virtual gel shows a smeary signal.

If an **HMW** (high molecular weight) smear is present, this is usually due to a problem with the fragmentation or sonication conditions.

- **Molarity Calculation**

Sequencers don’t take absolute quantities of DNA, they work on molarities. In PCR-free protocols, where the quantification is done through qPCR (see above), the molarity (in nM) is already obtained. If no qPCR was done, then the formula to calculate the molarity of a library is:
**Molarity (nM) = Concentration (ng/µL)×10<sup>6</sup> / Fragment length (bp) x 660**
This is assuming the result is in nM, where 660 is tha average molecular weight of a bp, the average fragment length is provided by the TapeStation/fragment analyzer, and the concentration is obtained from the mass provided by the Qubit quantification. 
Note: Always use the region tool in the TapeStation software to capture the entire smear, not just the highest peak, to get a true "average bp" for the formula.


 

