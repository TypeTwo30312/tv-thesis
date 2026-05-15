A streaming-write strategy based on \texttt{numpy.memmap} is used for the per-file accumulator together with a checkpoint every 500 peaks, because retaining all per-peak tensors in memory before writing exceeds the per-pod RAM allocation on the cluster for the larger BED files. % not relevant to the analysis, maybe a README inclusion


To characterize large-scale sequence-associated structure in Evo2 representations of 10 kb genomic regions surrounding transcription factor binding sites.


To what extent are Evo2 representations associated with local and large-scale sequence composition?
Does positional decomposition of Evo2 representations reveal structure not observable in globally pooled embeddings?
Do Evo2 representations reflect sequence-associated patterns consistent with the kilobase-scale compositional signals reported by Faltejsková et al.?

