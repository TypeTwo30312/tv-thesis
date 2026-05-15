``` latex

\chapter{Theory}
\label{sec:theory}

\section{Transcription factor binding}
\label{sec:tf-binding}

\subsection{Transcription factors and DNA binding}

Transcription factors (TFs) are proteins that bind DNA in a sequence-specific manner to modulate transcription.
They act through regulatory elements such as promoters, enhancers, and silencers, recruiting the transcription machinery or modulating its activity.
Most TFs share a modular architecture: a DNA-binding domain (DBD) that contacts the target sequence, paired with one or more effector domains responsible for transactivation, repression, or dimerisation.
Because the DBD determines binding specificity, TFs are classified by DBD family rather than by effector function.
The human genome encodes approximately 1600 sequence-specific TFs distributed across roughly 75 DBD families \cite{lambertHumanTranscriptionFactors2018}.
The families represented among the TFs studied in this thesis include the C2H2 zinc fingers (the largest human family, exemplified by CTCF), the basic helix-loop-helix proteins (MYC, which binds as an obligate heterodimer with MAX), and the ETS domain family (SPI1).

Target sequences are typically short motifs of 6--20~bp, recognized with tolerance for sequence variation.
Despite the considerable structural variety across DBD families, all sequence-specific TFs face the same fundamental problem:
they must locate their short target sequence among a large number of similar but non-functional sites distributed across a genome of several gigabases.

\subsection{Sequence determinants of binding}
\label{sec:readout}

The specificity of protein--DNA recognition rests on two complementary mechanisms: direct chemical recognition of bases (base readout) and recognition of sequence-dependent DNA geometry (shape readout).

Base readout operates through direct contacts -- hydrogen bonds and van der Waals interactions -- between amino acid side chains and the Watson--Crick edges of bases, predominantly in the major groove.
\fix{The major groove presents a distinctive pattern of hydrogen bond donors and acceptors} that permits discrimination between all four base pairs, including the orientation of A:T and T:A \cite{luscombeAminoAcidbaseInteractions2001, seemanSequencespecificRecognitionDouble1976}.
The C2H2 zinc finger illustrates the mechanism: each $\beta\beta\alpha$ module contacts approximately three base pairs through specificity residues on the recognition helix, and the contributions of successive modules combine to determine the affinity and specificity of the full array.
Position weight matrices (PWMs) are the natural statistical encoding of base readout, treating each position within the motif as an independent contribution to binding affinity (§\ref{sec:pwm}).

Shape readout depends on sequence context beyond the contact position.
The geometry at any given base pair -- minor groove width, propeller twist, helical twist, roll, and slide -- is influenced by neighbouring bases over several positions in either direction, and is read by protein contacts that respond to local shape and electrostatics rather than to specific bases.
The canonical example is recognition of narrow minor grooves by arginine residues: runs of A:T base pairs (A-tracts) produce locally narrowed minor grooves with concentrated negative electrostatic potential, which arginine side chains insert into and stabilise without forming base-specific hydrogen bonds \cite{rohsRoleDNAShape2009}.

The two mechanisms are genuinely complementary rather than redundant.
Crystal structures of TF--DNA complexes routinely show major-groove base-specific contacts and minor-groove shape-based contacts within the same interface, and augmenting PWM-based models with predicted shape features systematically improves binding prediction across TF families \cite{mathelierDNAShapeFeatures2016}.
Their combined reach is nonetheless bounded by the binding footprint: base readout depends on direct contacts within the 6--20~bp footprint, and shape readout, while sensitive to flanking sequence, is mediated through contacts within and immediately adjacent to it \cite{yangTranscriptionFactorFamilyspecific2017}.

\subsection{The specificity paradox}
\label{sec:paradox}

The degeneracy of TF motifs means that a typical 10~bp motif at moderate information content matches millions of sites in a mammalian genome, while in vivo occupancy -- assayed by ChIP-seq, the standard method described in §\ref{sec:chipseq} -- typically identifies between a few thousand and a few tens of thousands of bound sites per TF per cell line.
CTCF in K562 cells, for example, has approximately 52\,000 IDR-thresholded ENCODE peaks, against approximately 870\,000 matches to the canonical CTCF motif in the hg19 genome at standard FIMO thresholds \cite{dozmorovCTCFBioconductorData2022}.
This discrepancy between motif presence and actual occupancy has been termed the transcription factor specificity paradox; it was formalised for prokaryotic TFs as the futility theorem by \citet{wunderlichDifferentGeneRegulation2009}, who showed that in a sufficiently large genome even highly informative motifs predict more matches than there are TF molecules in the cell, making purely sequence-based recognition statistically insufficient.
In practice, the paradox is reflected in the weak correlation between intrinsic in vitro affinity (measured by HT-SELEX or protein-binding microarrays) and in vivo occupancy:
high-affinity sites can remain unbound, while low-affinity sites are often occupied \cite{slatteryAbsenceSimpleCode2014a}.

Several mechanisms partially resolve the discrepancy, each a substantial field in its own right.
Chromatin accessibility is regarded as the dominant factor: only a small fraction of the genome is accessible to DNA-binding proteins at any given time, the remainder occluded by nucleosomes or compacted into heterochromatin, and most in vivo TF binding occurs within accessible regions \cite{thurmanAccessibleChromatinLandscape2012}.
Cooperative binding with cofactors restricts binding further: many TFs bind productively only as part of a complex -- homodimers, heterodimers, or larger assemblies -- so the combinatorics of cofactor co-occupancy limit where productive binding can occur \cite{sonmezerMolecularCooccupancyIdentifies2021}.
Cellular context adds a third layer: TF abundance, post-translational modification, and subcellular localisation are all condition-dependent and regulate the effective concentration of binding-competent protein \cite{kribelbauerLowAffinityBindingSites2019}.

A fourth contribution, less studied than the preceding three, is sequence context beyond the motif itself.
\citet{drorWidespreadRoleMotif2015} demonstrated, in a systematic analysis of 239 TFs from HT-SELEX and 56 from ChIP-seq data, that the sequence and DNA shape environment (termed "homotypic environment", for its typical resemblance of the motifs composition) surrounding bound motifs differs significantly from that surrounding unbound occurrences of the same motif, with differences extending well beyond the core binding site (up to 300\,bp) and exhibiting TF-family-specific patterns.
Crucially, the signal was present in both the in vitro and in vivo datasets, ruling out the cellular environment as its sole source and establishing flanking sequence context as a genuine, sequence-encoded contributor to binding discrimination.
Unlike chromatin accessibility and cooperative binding, this contribution is in principle directly accessible to any model that operates on raw sequence input.

The spatial scale at which sequence context influences binding spans several orders of magnitude: from the immediate flanks of the core motif \cite{drorWidespreadRoleMotif2015}, through the tens-to-hundreds of base pairs over which sequence composition modulates the nonspecific affinity experienced by a scanning TF \cite{afekGenomewideOrganizationEukaryotic2013, selaDNASequenceCorrelations2011}, \fix{out to the kilobase-scale compositional gradients recently reported by \citet{faltejskovaNonconsensusFlankingSequence2026a}(preprint).}
The findings of this last study are the biological motivation for the analyses in this thesis and are discussed in §\ref{sec:faltejskova}.

\section{Next-generation sequencing and ChIP-seq}
\label{sec:chipseq}

\subsection{Short-read sequencing}

Next-generation sequencing (NGS) refers to high-throughput DNA sequencing technologies that determine the nucleotide sequence of millions to billions of short DNA fragments in parallel.
The dominant platform in current use is Illumina sequencing-by-synthesis:
libraries of DNA fragments are clonally amplified on a flow cell surface, and each cycle of synthesis incorporates fluorescently labelled reversible-terminator nucleotides whose identity is read by imaging \cite{bentleyAccurateWholeHuman2008}.
Typical read lengths are 50--150~bp per end; paired-end protocols sequence both ends of each fragment and are standard for ChIP-seq.
The output is a set of FASTQ files containing read sequences and per-base quality scores.
Key limitations are the short read length, which complicates mapping to repetitive regions,
and a characteristic per-base error rate of approximately $10^{-3}$, dominated by substitution errors.

NGS functions as a platform on which different upstream wet-lab protocols measure different genomic quantities:
RNA-seq measures transcript abundance, ATAC-seq measures chromatin accessibility, and ChIP-seq measures protein--DNA contacts.
The platform is otherwise identical across assays; what changes is the biological molecule subjected to fragmentation and \fix{sequencing}.

\subsection{Chromatin immunoprecipitation sequencing}

Chromatin immunoprecipitation followed by sequencing (ChIP-seq) is the standard method for mapping genome-wide protein--DNA contacts at the population level \cite{baileyPracticalGuidelinesComprehensive2013}.
In the crosslinking variant used for most TF ChIP-seq experiments, cells are treated with formaldehyde to covalently bind protein--DNA complexes in situ, chromatin is then fragmented by sonication or enzymatic digestion to a target size of approximately 200~bp, and an antibody specific to the TF of interest is used to immunoprecipitate the crosslinked complexes.
After reversal of crosslinks, the co-purified DNA is sequenced.
The resulting reads are enriched at genomic positions occupied by the protein, producing a coverage signal that is higher at bound sites than at unbound genomic background.

The output of ChIP-seq is interpretable as a population-averaged estimate of contact frequency:
sites occupied in a large fraction of cells contribute more fragments to the library and produce sharper, higher-amplitude peaks.
This population averaging is a fundamental limitation: the assay does not report single-cell occupancy, cannot distinguish direct from indirect binding (a TF pulled down as part of a complex is not directly bound to the immunoprecipitated DNA), and is sensitive to antibody quality in ways that are difficult to control post-hoc.

\subsection{Cut\&Tag as a methodological contrast}

\ask{probably cut}
An alternative to ChIP-seq that is increasingly used for TF mapping is CUT\&Tag \cite{kaya-okurCUTTagEfficientEpigenomic2019}.
Rather than immunoprecipitating crosslinked complexes, CUT\&Tag uses a Protein~A--Tn5 transposase fusion, directed to the protein of interest by an antibody, to tagment (cut and insert sequencing adaptors into) the DNA in the immediate vicinity of the bound protein.
The method requires lower cell inputs, produces lower background, and generates sharper peaks than ChIP-seq, making it particularly attractive for TFs with weak or transient binding.

\subsection{Read alignment and peak calling}

Sequencing reads are aligned to the reference genome using short-read aligners such as BWA \cite{liAligningSequenceReads2013} or Bowtie~2 \cite{langmeadFastGappedreadAlignment2012}, producing coordinate-sorted BAM files with alignment information and mapping quality scores.
Reads mapping to multiple locations are typically filtered; duplicate reads arising from PCR amplification of the same fragment are also marked and excluded. \ask{can i cite Bowtie 2, if i don't have access to the paper?}

Enriched genomic regions (peaks) are identified by peak-calling algorithms that compare local fragment density in the immunoprecipitated sample against a control library, usually input chromatin from the same cells.
MACS2 is the standard algorithm for TF ChIP-seq \cite{zhangModelbasedAnalysisChIPSeq2008};
it models the expected fragment distribution, estimates local background from a sliding window, and assigns a $p$-value and $q$-value to each candidate peak under a Poisson model.
The primary output is a narrowPeak BED file containing one row per peak, with columns for chromosomal coordinates, a peak score, $-\log_{10}(p)$, $-\log_{10}(q)$, and the offset of the peak summit from the start coordinate.
For TF ChIP-seq, peak widths are typically 200--500~bp.

\subsection{Reproducibility filtering and the ENCODE project}
\label{sec:encode-idr}

The Encyclopedia of DNA Elements (ENCODE) project provides a systematic, large-scale catalogue of functional genomic elements through standardised application of high-throughput assays including ChIP-seq, ATAC-seq, and RNA-seq \cite{dunhamIntegratedEncyclopediaDNA2012}.
ENCODE enforces uniform experimental protocols, antibody validation requirements, and bioinformatic pipelines, and applies automated quality-control audits to all deposited experiments; experiments flagged by these audits are marked on the portal and excluded from downstream use.
All ChIP-seq data used in this thesis were obtained from the ENCODE portal.\footnote{\url{https://www.encodeproject.org}}

Because ChIP-seq is sensitive to antibody efficiency and library complexity, peaks called from a single experiment may include irreproducible signals.
The Irreproducible Discovery Rate (IDR) framework provides a principled way to rank peaks by their reproducibility across biological or technical replicates \cite{liMeasuringReproducibilityHighthroughput2011}.
IDR fits a mixture model to the joint rank distribution of peaks across two replicates and assigns each peak a probability of arising from the reproducible component of the mixture.
Peaks below an IDR threshold are retained; those above are discarded as likely irreproducible.
ENCODE distributes IDR-thresholded peak calls as the recommended file type for downstream analysis \cite{dunhamIntegratedEncyclopediaDNA2012};
conservative IDR-thresholded calls, thresholded at an IDR of 0.05 applied to both the self-consistency and pooled-consistency ratios, are preferred where available.
All peak files used in this thesis are conservative IDR-thresholded narrowPeak files.

\subsection{Long-range sequence context around binding sites}
\label{sec:faltejskova}

The ChIP-seq peak calls distributed by ENCODE identify the genomic coordinates of protein--DNA contacts, but the narrowPeak format records only a ~200--500~bp window around the occupied site.
Whether the broader sequence context surrounding a bound site encodes systematic, non-random features -- and whether those features are consistent with a role in guiding TF recognition -- is a question that cannot be answered from the peak coordinates alone.

\citet{faltejskovaNonconsensusFlankingSequence2026a} addressed this question in a recent descriptive study. % currently in preprint
They analysed a $\pm$5000~bp window around in vivo binding sites for ten TFs drawn from ENCODE ChIP-seq and one CUT\&Tag dataset, dividing each 10~kb window into 50~bp non-overlapping segments and computing sequence composition features -- GC content, dinucleotide frequencies, predicted DNA shape (via deepDNAshape), and HOCOMOCO-based TF affinity scores -- across the profile from distal flanks to peak centre. % so cite deepDNAshape

The principal finding is that GC content is significantly elevated in a patch spanning approximately 1--2~kb around the binding site for most of the ten TFs examined, with a typical magnitude of~$\sim$10 percentage points above the distal flank.
Directional asymmetries in dinucleotide composition, pointing toward the binding site, were observed for several TFs including MYC, FOXK2, and p53, and are absent or weak for CTCF and SPI1.
DNA shape features predicted from the local sequence context -- in particular roll and helical twist -- change in a manner consistent with increased bendability near the binding site.
A small but significant enrichment of predicted Z-DNA and G-quadruplex motifs at the binding site was also observed for selected TFs, though the authors note these are insufficient to account for the full dinucleotide pattern.

These findings are interpreted through a ``statistical funnel'' model: broad compositional gradients around binding sites create a low-free-energy landscape that biases TFs undergoing one-dimensional facilitated diffusion toward the target site, complementing the short-range specific recognition described in §\ref{sec:readout}.
The study explicitly characterises this model as hypothesis-generating.
Critically, it is a descriptive analysis: no model is tested for whether it can detect or exploit the sequence gradients described.

This descriptive gap is the primary motivation for the present thesis.
The sequence features characterised by \citet{faltejskovaNonconsensusFlankingSequence2026a} extend over 1--2~kb on either side of the binding site -- a spatial scale that is inaccessible to conventional motif-based models by design, but that falls within the context window of recent genomic foundation models operating at single-nucleotide resolution.
Whether the per-token embeddings of such a model encode the kilobase-scale compositional structure described above is the empirical question taken up in this thesis.
The foundation model used, Evo~2, is introduced in §\ref{sec:foundation-models}.

\section{Computational approaches to TF binding site analysis}
\label{sec:pwm}

\subsection{Position weight matrices}

The dominant computational representation of TF binding specificity is the position weight matrix (PWM), \fix{also called a position-specific scoring matrix (PSSM).}
A PWM is derived from a set of aligned binding sequences of fixed width $L$:
at each position $i \in \{1, \ldots, L\}$ and nucleotide $b \in \{A, C, G, T\}$, the log-odds score $w_{i,b} = \log_2 (f_{i,b} / p_b)$ is computed, where $f_{i,b}$ is the observed frequency of nucleotide $b$ at position $i$ among the training sequences and $p_b$ is the background frequency of $b$ \cite{stormoDNABindingSites2000}.
The score of a candidate sequence is the sum of per-position log-odds terms;
a threshold on this score determines whether the sequence is called a binding site.
The information content of the matrix, $\mathrm{IC} = \sum_{i} \sum_b f_{i,b} \log_2(f_{i,b}/p_b)$, is a measure of the specificity of the motif and is used to produce the familiar sequence logo visualisation \cite{schneiderSequenceLogosNew1990}.
\todo{sequence logo visual}

The PWM model rests on an independence assumption: the contribution of each position to the total score is treated as additive, with no coupling between positions.
This assumption is biologically inaccurate -- adjacent nucleotides are correlated in both sequence and structural terms -- but the model is computationally tractable, transferable across organisms, and sufficiently accurate for many applications.
It remains the standard representation in curated databases such as JASPAR \cite{ovekbaydarJASPAR2026Expansion2026} and HOCOMOCO \cite{vorontsovHOCOMOCO2024Rebuild2024}, which together provide PWM models for the large majority of human TFs.

\subsection{Motif discovery and enrichment analysis}

Given a set of ChIP-seq peak sequences, two related tasks arise:
discovering the sequence motif enriched in the peaks (motif discovery), and testing whether a known motif is significantly enriched relative to a background set (motif enrichment).

MEME (Multiple Em for Motif Elicitation) addresses the discovery task using expectation maximisation to identify the PWM that best explains an overrepresented pattern across the input sequences \cite{baileyUnsupervisedLearningMultiple1995};
the broader MEME Suite extends this to enrichment testing, scanning, and comparative analysis \cite{baileyMEMESuite2015}.
HOMER (Hypergeometric Optimization of Motif EnRichment) is widely used for ChIP-seq specifically:
it optimises motif enrichment relative to a GC-matched background set, making it robust to the GC bias that is characteristic of many TF binding sites \cite{heinzSimpleCombinationsLineageDetermining2010}.
Scanning a genome with a known PWM to identify all sites above a score threshold is performed by tools such as FIMO \cite{grantFIMOScanningOccurrences2011};
this is the operation used to count the motif matches discussed in §\ref{sec:paradox}.

\subsection{Extensions beyond the independence model}

Several approaches have been developed to relax the positional independence assumption.
Dinucleotide weight matrices (DWMs) model pairwise dependencies between adjacent positions, capturing the nearest-neighbour sequence correlations that are mechanistically important for
shape readout \cite{bulykNucleotidesTranscriptionFactor2002}.
Hidden Markov models and related probabilistic graphical models can represent variable-length motifs with internal positional dependencies, at the cost of increased model complexity and training data requirements \cite{weirauchEvaluationMethodsModeling2013}.
Convolutional neural networks trained on ChIP-seq data learn sequence representations that implicitly capture multi-position dependencies and outperform PWMs on held-out binding
prediction benchmarks \cite{alipanahiPredictingSequenceSpecificities2015};
however, such models are typically trained in a supervised setting and require labelled positive and negative sequences.

A complementary line of work incorporates predicted DNA shape features alongside sequence features.
\citet{mathelierDNAShapeFeatures2016} showed that adding per-position shape predictions (minor groove width, propeller twist, roll, and helical twist computed by DNAshape) as features alongside the PWM score systematically improves in vivo binding prediction across a broad range of TF families.
The deepDNAshape model \cite{liPredictingDNAStructure2024}, which predicts the same structural features from sequence using a neural network trained on molecular dynamics simulations, is used by \citet{faltejskovaNonconsensusFlankingSequence2026a} to characterise shape changes across the 10~kb window around binding sites.

\subsection{The kilobase-scale context gap}
\label{sec:motif-gap}

All methods described above share a fundamental constraint:
they operate on a fixed window whose width is determined by the binding footprint of the TF.
PWMs typically span 6--20~bp; even extended models incorporating shape features or positional dependencies are anchored to the core motif and its immediate neighbourhood, typically within 50~bp.
By construction, these methods cannot represent the kilobase-scale sequence composition gradients described in §\ref{sec:faltejskova}, because they have no mechanism for incorporating sequence at that distance from the binding site.

This constraint is not a limitation of any specific algorithm but is inherent to the class of models:
the binding site itself is defined as a short sequence, and the model is designed to score that sequence.
Capturing kilobase-scale context requires a qualitatively different modelling approach -- one that processes the full sequence window and produces representations that integrate information across its entire length.
Genomic foundation models, described in the following section, are a natural candidate for this role.

\section{Genomic foundation models}
\label{sec:foundation-models}

\subsection{Self-supervised pretraining and the foundation model paradigm}

Foundation model, a relatively recent term, describes a machine learning model that in general utilises self-supervised pretraining on large unlabelled datasets, with the intention of subsequent task-specific adaptation by fine-tuning or distillation.
In self-supervised learning, the model is trained to predict held-out portions of the input sequence from the remaining context: in masked language modelling, randomly selected tokens are replaced with a mask symbol and the model is trained to recover them; in autoregressive modelling, the model is trained to predict each token from all preceding tokens.
Neither objective requires labelled data;
the supervision signal is derived entirely from the sequence itself.
The resulting model learns distributed representations that encode statistical regularities of the training data and that can be transferred to downstream tasks by fine-tuning on small labelled datasets or by extracting intermediate-layer embeddings for use as features \cite{bommasaniOpportunitiesRisksFoundation2022}.

The transformer architecture \cite{vaswaniAttentionAllYou2017} is the backbone of most current foundation models.
Its core mechanism, scaled dot-product attention, computes pairwise interactions between all positions in the input sequence, allowing each position's representation to integrate information from arbitrarily distant context.
This property makes transformers well-suited for tasks that require long-range dependencies; however, the computational and memory cost of full attention scales quadratically with sequence length, which places a practical ceiling on the context window at training time.
For DNA, where biologically relevant interactions can span thousands to millions of base pairs, this is a fundamental constraint.

\subsection{Genomic foundation models: a brief landscape}

The first genomic foundation model applied to regulatory sequence was DNABERT \cite{jiDNABERTPretrainedBidirectional2021a}, which adapted the BERT masked language modelling objective to the human reference genome.
DNABERT tokenises the genome into overlapping $k$-mers (default $k = 6$) and pretrains a bidirectional transformer on the resulting token sequences.
The pretrained model was shown to transfer to promoter prediction, splice site detection, and TF binding site prediction by fine-tuning on small labelled datasets.
A limitation of the $k$-mer tokenisation is that it conflates sequence position with token identity and produces a highly redundant vocabulary;
the context window at inference time was also limited to 512 tokens, corresponding to roughly 512~bp of genomic sequence.

The Nucleotide Transformer \cite{dalla-torreNucleotideTransformerBuilding2025} scaled the masked language modelling approach to larger model sizes (up to 2.5~billion parameters) and a multi-species training dataset.
Its context window remains limited to a about 6\,kbp at inference.

HyenaDNA \cite{nguyenHyenaDNALongRangeGenomic2023} addressed the context length problem by
replacing the attention mechanism with the Hyena operator, a data-controlled convolutional
recurrence that scales linearly in sequence length.
Trained autoregressively on the human reference genome at single-nucleotide resolution,
HyenaDNA extended the practical context window to up to 1~million base pairs, demonstrating
for the first time that a genomic foundation model could be trained and evaluated at the scale
of entire chromosomal regions.
Performance on short-range benchmarks was competitive with transformer-based models;
the primary advance was the demonstration that linear-scaling architectures could handle
genomic context lengths that are infeasible for full attention.

\subsection{Evo 1 and Evo 2}
\label{sec:evo2bg}

The Evo model \cite{nguyenSequenceModelingDesign2024} extended the long-context autoregressive
approach to a multi-kingdom training dataset, pretraining a 7~billion parameter model on
prokaryotic, viral, and eukaryotic genomes using the StripedHyena architecture.
StripedHyena interleaves multi-head attention layers with Hyena convolutional layers,
combining the strong local feature extraction of attention with the linear-scaling long-range
context capture of the Hyena operator.
Evo demonstrated capabilities in zero-shot variant effect prediction, coding sequence
generation, and CRISPR guide RNA design.

Evo~2 \cite{brixiGenomeModellingDesign2026} is the successor model, published in
\emph{Nature} in 2026.
It is trained autoregressively on OpenGenome2, a curated dataset of 8.8~trillion nucleotides drawn from organisms across all domains of life, including prokaryotes, archaea, and
eukaryotes.
The architecture is StripedHyena~2, an updated version that further improves the integration
of attention and gated convolutional layers.
Models are available in three sizes:
1~billion, 7~billion, and 40~billion parameters.
The 7~billion parameter checkpoint used in this thesis operates at single-nucleotide
resolution with a context window of up to 1~million base pairs in its extended-context
configuration (\texttt{evo2\_7b}), or 8192 tokens in its base configuration
(\texttt{evo2\_7b\_base}).
Because the 10~kb windows used in this thesis exceed the 8192-token limit of the base
checkpoint, the extended-context \texttt{evo2\_7b} checkpoint is used throughout. \ask{this could stay or move to methods...}

Evo~2 has been benchmarked on variant effect prediction across coding and non-coding variants,
zero-shot fitness prediction for proteins and ncRNA, and generative sequence design;
these evaluations are described in the original publication.
\ask{Whether and how Evo~2 embeddings represent the sequence context around in vivo TF binding
sites has not been examined.} % nope, this should be description of what they did with TFs

\subsection{Embedding extraction, binning, and normalisation}
\label{sec:embedding-extraction}

Foundation models trained autoregressively produce, at each token position, a hidden-state
vector whose dimensionality equals the model's embedding dimension.
These per-token hidden states are the primary substrate for embedding-based analyses:
they can be mean-pooled across positions to yield a single fixed-length representation of the
whole sequence, or retained at full resolution to preserve positional information.

For the 7~billion parameter Evo~2 checkpoint, the embedding dimension is 4096.
Each nucleotide at each sequence position produces a 4096-dimensional vector;
a 10~kb sequence therefore yields a $(10{,}000 \times 4096)$ matrix of hidden states before
any pooling.
Each row of this matrix can be thought of as a point in a 4096-dimensional space,
characterised by both a \emph{magnitude} -- the Euclidean length of the vector -- and a
\emph{direction} -- the pattern of relative activations across the 4096 dimensions.

The choice of which layer's hidden states to extract is consequential.
The final layer is specialised for the next-token prediction objective and may carry
information optimised for that task rather than for general-purpose sequence representation.
Intermediate layers have been shown empirically to yield more transferable representations
for downstream tasks;
\fix{the exon classifier distributed with Evo~2 uses layer \texttt{blocks.28.mlp.l3} as the
embedding source}, and this layer is used throughout the present thesis for the same reason.

In this thesis, each 10~kb sequence is processed in a single forward pass through
\texttt{evo2\_7b}, and the hidden states from \texttt{blocks.28.mlp.l3} are mean-pooled
within non-overlapping 50~bp bins to yield a $(200 \times 4096)$ positional embedding
profile per peak.
This binning resolution matches the 50~bp window size used by
\citet{faltejskovaNonconsensusFlankingSequence2026a} for hand-engineered feature extraction,
allowing a direct spatial correspondence between the two analyses.

\subsubsection{Magnitude and directional variation}

When embeddings are compared based on cosine siimilarity or subjected to principal component analysis, the dominant axis of variation is typically the magnitude of the hidden-state  vectors.
Magnitude reflects the overall scale of model activations and may co-vary with sequence properties such as GC content, but it conflates many sources of variation and can obscure subtler structure in the data.

To examine the directional component of embedding variation independently of magnitude, each embedding vector can be \emph{L2-normalised}: divided by its own Euclidean norm,
\begin{equation}
  \hat{\mathbf{v}} = \frac{\mathbf{v}}{\|\mathbf{v}\|_2},
\end{equation}
so that the resulting vector $\hat{\mathbf{v}}$ lies on the unit hypersphere and has norm exactly one (\ref{figsomething}).
After L2 normalisation, all vectors have the same length and differ only in direction;
a subsequent PCA therefore responds exclusively to directional differences in the embedding space, with magnitude variation removed by construction.
The practical motivation is that magnitude and direction may carry distinct biological information, and separating them allows each to be interpreted on its own terms.
The analyses in §\ref{sec:results-block2b} apply this normalisation to the file-mean positional profiles to characterise what directional structure the embeddings contain after magnitude is removed.


\subsection{Autoregressive context and its implications}

A property of autoregressive models that is directly relevant to the interpretation of
positional embedding profiles is the asymmetry of context:
at any token position $t$, the hidden state is computed from all preceding tokens
$1, \ldots, t-1$ but has no access to tokens at positions $t+1, \ldots, L$.
Tokens near the beginning of the sequence therefore have little or no left context, and their
representations may differ systematically from tokens in the interior of the sequence where
left context is rich.
This warm-up effect is a model-level property, not a sequence-biological signal, and must be
accounted for when interpreting positional embedding profiles derived from fixed-start 10~kb
windows.
The practical consequence for this thesis -- and the trimming strategy adopted to address it
-- is described in §\ref{sec:results-block2}. % this may not be relevant, depending on the results, candidate for drop


```