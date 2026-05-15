``` latex

\chapter{Aims}
\label{sec:aims}

A recent preprint by \citet{faltejskovaNonconsensusFlankingSequence2026a} reports that
the sequence composition around in vivo transcription factor binding sites is non-random
at a spatial scale of 1--2~kb on either side of the binding site, with TF-specific
patterns in GC content and directional dinucleotide asymmetry extending well beyond the
core binding footprint.
These findings raise the question of whether such large-scale compositional structure is
reflected in the internal representations of a genomic foundation model operating at the
same spatial scale.
This thesis addresses that question descriptively, using Evo~2 \cite{brixiGenomeModellingDesign2026}
as the model and ChIP-seq peak sets from ENCODE as the biological substrate.

The specific aims are:

\begin{enumerate}
  \item To characterise the extent to which Evo~2 embeddings are associated with
    sequence composition across 10~kb windows centred on transcription factor binding
    sites.
  \item To determine whether positional structure in Evo~2 embeddings across the 10~kb
    window differs from what is visible in globally pooled representations.
  \item To assess whether variation in the directional component of Evo~2 embeddings
    is consistent with the kilobase-scale compositional signals described by
    \citet{faltejskovaNonconsensusFlankingSequence2026a}.
\end{enumerate}

```