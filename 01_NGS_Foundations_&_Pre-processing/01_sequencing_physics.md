# Understanding the sequencing process

## NGS VS Sanger Sequencing
  
The key innovation that transforms DNA replication into the DNA-sequencing strategy at the core of both Sanger and NGS is the use of **unextendable, fluorescently labeled modified bases**. There are four different colors of modified bases for A, T, G, and C. In Sanger sequencing, only a small percentage of bases are modified (dideoxynucleotides, ddNTPs), whereas in Illumina´s techonologies, all available bases are modified with reversible 3´blockers. In both sequencing techniques, when polymerase incorporates a modified base into the copied strand, extension of the new strand stops and, critically, this newly terminated strand is uniquely colored to reflect its most recently added base.

The fundamental challenge for the sequencer, then, is to organize molecules such that their fluorescence signal is interpretable. In Sanger sequencing, an ensemble of DNA molecules — all originating from the same position on the template but having different size due to termination at different positions — are arranged in an electric field by capillary electrophoresis, which separates them by size because DNA is negatively charged. As the molecules migrate in the presence of the electric field, they flow past a detector that registers the fluorescence intensity and color, yielding a series of peaks that can be mapped directly to a DNA sequence.

While Sanger relies on physical separation by size, the leap to NGS was made possible by **massively parallel sequencing**, where millions of unique clusters are sequenced simultaneously on a solid surface (the flow cell).

| Sanger | NGS |
|--------|-----|
| 1 fragment at a time | Millions of fragments |
| Slow | Fast |
| Limited throughput | Detects small changes |

## Illumina sequencing

Illumina sequencers use a sequencing method called **sequencing by synthesis**. In this method, DNA strands bind to oligonucleotides on a glass slide that match the adapter sequence, and remain fixed at the same position throughout the entire sequencing reaction. For this to happen, the library has to first be denatured. 
Once bound, the DNA fragment (or forward strand) now serves as a template for the extension of the oligonucleotide. After extension, the forward strand is released and washed away.
The next step is the amplification of the generated fragments. Since these were generated from DNA containing adapters, the adapter on the other end can now bend (“bridge”) and bind another complementary oligonucleotide, which involves the bending of the DNA molecule. Once bound, the chain is extended and both chains are denatured, in a process that is repeated over and over, generating a cluster of the same sequence. Importantly, longer fragments (700-800 bp) are stiffer and therefore have more problems bending.
The reverse strands get cut (thanks to a specific sequence in the adapter) and washed away.
Primers are added together with nucleotides that have a specific fluorophor for each of them and a terminator that prevents the chain from being further extended. The camera detects the fluorescent signal, the terminator is washed away, and the next base is sequenced. This process repeats generating **reads**. If the "cleaving" enzyme doesn't work perfectly to wash away the terminator, the next base can't bind, so this read will be one nucleotide behind. On the other hand, two nucleotides can accidentally be incorporated in one cycle because the "terminator" was missing or broken, so now this read will be one nucleotide ahead. These are called **phasing** and **pre-phasing**, respectively. As the run goes on (e.g., cycle 100, 150), these "out of sync" strands accumulate. This is why Quality Scores (Q30) always drop toward the end of a read. When they drop too early, it is usually because of a chemistry or hardware issue (clogged fluidics or old reagents), and the expiry date of the reagents or the pump performance need to be checked.
In the case of **paired-end sequencing**, where fragments are read from both ends, as opposed to single-end sequencing, the process described before is repeated, but in this case the forward strand is cleaved, leaving the reverse one, which will be read as previously described. Paired-end sequencing, although more expensive, provides much higher confidence in the sequence prediction and is therefore the usual go-to choice for Illumina sequencing.
Since the adapters contain indexers, that will allow the sequencer to determine which reads belong to each sample in a process called demultiplexing. The sequencer reads the index as a separate, tiny sequencing run (usually 8 or 10 bases) called an Index Read. If the index read has low quality, the machine won't know which sample the DNA belongs to. These reads end up in an **undetermined** folder. 


