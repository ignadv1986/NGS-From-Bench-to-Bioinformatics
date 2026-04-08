# Understanding the sequencing process

## NGS VS Sanger Sequencing
  
The key innovation that transforms DNA replication into the DNA-sequencing strategy at the core of both Sanger and NGS is the use of **unextendable, fluorescently labeled modified bases**. In Sanger sequencing, only a small percentage of bases are modified (dideoxynucleotides, ddNTPs), causing random termination of fragments, whereas in Illumina technologies, all available bases are modified with reversible 3´blockers. In both sequencing techniques, when polymerase incorporates a modified base into the copied strand, extension of the new strand stops and, critically, this newly terminated strand is uniquely colored to reflect its most recently added base. The fundamental challenge for the sequencer, then, is to organize molecules such that their fluorescence signal is interpretable.

In Sanger sequencing, an ensemble of DNA molecules — all originating from the same position on the template but having different size due to termination at different positions — are arranged in an electric field by capillary electrophoresis, which separates them by size because DNA is negatively charged. As the molecules migrate in the presence of the electric field, they flow past a detector that registers the fluorescence intensity and color, yielding a series of peaks that can be mapped directly to a DNA sequence.

While Sanger relies on physical separation by size, the leap to NGS was made possible by **massively parallel sequencing**, where millions of unique clusters are sequenced simultaneously on a solid surface (the flow cell). Because the blockers on the nucleotides are reversible, Illumina sequencers can register the fluorescence of the added nucleotide, remove the terminator, and repeat over and over again to generate the complete reads.

| Feature | Sanger | NGS |
| :--- | :--- | :--- |
| **Throughput** | One fragment per capillary | Millions of fragments |
| **Separation** | Physical (capillary electrophoresis) | Spatial (fixed position on a flow cell) |
| **Output** | Long reads (up to 1kb), low volume | Short reads (50-300bp), massive volume |
| **Primary use** | Single gene validation, plasmid sequencing | Genomes, Transcriptomes, Epigenomes |

## Illumina sequencing

Illumina sequencers use a sequencing method called **sequencing by synthesis**. In this method, DNA strands bind to oligonucleotides on a glass slide that match the adapter sequence, and remain fixed at the same position throughout the entire sequencing reaction. For this to happen, the DNA that has been processed for sequencing, or **library** (see the [next section](./02_library_preparation.md) of this repository for more information on library preparation) has to first be denatured. 
Once bound, the DNA fragment (or forward strand) serves as a template for the extension of the oligonucleotide. 

### Bridge Amplification and Cluster Generation

After the initial extension, the original strand is washed away. The remaining strand bends over to **bridge** and bind a complementary oligonucleotide, serving as a template for the extension of the latter. Through repeated cycles of denaturing and extension, a monoclonal **cluster** (thousands of copies of the same DNA molecule) is formed in a single spot. Importantly, longer fragments (700-800 bp) are stiffer and therefore have more problems bending, leading to poor cluster generation and a bias toward shorter fragments in the final data.
The reverse strands get cut (thanks to a specific sequence in the adapter) and washed away, and only copies of the strands that originally bound the flow cell are left.

### Reading the Fragments

Primers are added together with nucleotides that have a specific fluorophor for each of them and a terminator that prevents the chain from being further extended. The sequencer excites the fluorophores with a laser and the camera detects the fluorescent signal. The terminator is then washed away, and the process is repeated a determined number of times (**read length**). This leads to the generation of **reads**.

### Paired-end sequencing

In the case of **paired-end sequencing**, where fragments are read from both ends, as opposed to single-end sequencing, the process described before is repeated, but in this case the forward strand is cleaved, leaving the reverse one, which will be read as previously described. Paired-end sequencing, although more expensive, provides much higher confidence in the sequence prediction, making it the usual go-to choice for Illumina sequencing.

### Optical chemistry

As explained above, the sequencer identifies bases by exciting fluorophores with a laser and capturing an image. To increase sequencing speed and reduce the cost of the hardware, Illumina evolved the imaging chemistry from four colors down to two:

- **4-channel chemistry** (e.g., MiSeq, HiSeq): Each of the four bases has a unique dye and is captured in four separate images per cycle. While highly accurate, the time required for four imaging steps per cycle limits throughput.
- **2-channel chemistry** (e.g., NextSeq, NovaSeq): only two dyes are used and the sequencer takes two pictures, one through a red filter and one through a green filter. Each nucleotide bound to one (red for C and green for G), 2 (red and green for T, resulting in yellow signal), or none (A, the sequencer registers no fluorescence in that position and registers it as an A).

## Patterned Flow Cells

In earlier systems (MiSeq, HiSeq), DNA fragments bound to the flow cell at random. As bridge amplification occurred, clusters could grow too close to each other, leading to fluorescence overlapping, and thereby causing the camera to fail to distinguish the two signals. To overcome this challenge, a Poisson distribution was employed: a limiting dilution of sample was used, reducing the possibility that two different DNA fragments "landed" too close. However, this also leads to a big part of the flow cell being empty, limiting sequencing efficiency.

Modern high-throughput machines (NovaSeq) use **patterned flow cells** where billions of nanowells are distributed on the glass at exact intervals. This allows the sequencer to know exactly where every cluster should be, maximizing the number of reads that can be obtained from single slide. To ensure each nanowell contains only one unique DNA fragment, Illumina uses **ExAmp kinetics**. The reagents are designed so that the moment the first DNA molecule enters a well, it replicates so violently fast that it fills the entire volume of the well instantly. This high-speed cloning physically crowds out and excludes any other DNA molecules from entering.

## Technical Challenges

### Phasing and pre-phasing

If the "cleaving" enzyme doesn't work perfectly to wash away the terminator, the next base won't be able to bind, so this read will be one nucleotide behind. On the other hand, two nucleotides can accidentally be incorporated in one cycle because the "terminator" was missing or broken, so now this read will be one nucleotide ahead. These are called **phasing** and **pre-phasing**, respectively. As the run goes on (e.g., cycle 100, 150), these "out of sync" strands accumulate. This is why Quality Scores (Q30) always drop toward the end of a read (see the [sequencing QC](./03_sequencing_QC.md) section of this repository). When they drop too early, it is usually because of a chemistry or hardware issue (clogged fluidics or old reagents), and the expiry date of the reagents or the pump performance need to be checked.

### Low quality index read

The adapters bound to the genomic DNA contain **indexers**: these are short sequences (usually 8 or 10 bases), different for each sample. Thanks to their presence, all samples can be sequenced together, and the sequencer can determine which reads belong to each sample in a process called **demultiplexing**.

Importantly, the sequencer reads the index as a separate, tiny sequencing run called an **index read**. If the index read has low quality, the machine won't know which sample the DNA belongs to, and these reads end up in an **undetermined** folder. High percentages of undetermined reads usually point to library prep errors or issues with the sample sheet.

### Index hopping

Because ExAmp is a high-energy chemical reaction and wells are so close together, any leftover free adapters in the library can accidentally prime a reaction in a neighboring well. This index hopping can lead to crosstalk where reads are assigned to the wrong sample. This is why modern protocols require **Unique Dual Indexes (UDIs)**, a digital safety net for the high-speed chemistry.

