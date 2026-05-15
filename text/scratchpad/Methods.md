``` latex


\chapter{Methods}

\section{Data}

\subsection{Transcription factors}

The descriptive analysis is performed on three human transcription factors selected to span distinct binding modes relevant to the questions posed in this thesis: CTCF, MYC, and SPI1.
CTCF is an eleven-zinc-finger architectural factor with a long, well-characterised consensus motif and broad genomic occupancy, included as a strong-motif baseline against which embedding-based representations can be evaluated \cite{ongCTCFArchitecturalProtein2014}
MYC is a basic helix-loop-helix leucine-zipper factor that, in the analysis of \citet{faltejskovaNonconsensusFlankingSequence2026a}, displays a pronounced directional dinucleotide asymmetry in the kilobase-scale flanks of its in vivo binding sites, making it a natural target for testing whether long-range sequence structure is reflected in Evo~2 representations \cite{luscherRegulationGeneTranscription2012a}.
SPI1 (PU.1) is an ETS-family lineage-specific factor that does not exhibit the directional flanking signature reported for MYC and is therefore included as a contrast factor for which the kilobase-scale signal is expected to be weaker or absent \cite{rothenbergMechanismsActionHematopoietic2019}.

\subsection{ChIP-seq data sources}
\label{sec:chip-data} % check that

For each transcription factor, three ChIP-seq experiments are obtained from the ENCODE portal \cite{dunhamIntegratedEncyclopediaDNA2012}, yielding nine BED files in total.
Conservative IDR-thresholded narrowPeak files \cite{liMeasuringReproducibilityHighthroughput2011} are used where available; otherwise, the IDR-thresholded peak file is used.
Cell-line selection follows the inclusion criterion adopted by \citet{faltejskovaNonconsensusFlankingSequence2026a}, which retains only cell lines appearing in at least three experiments across their panel.
The accessions, transcription factors, and biosamples comprising the embedded set are listed in Table~\ref{tab:bedfiles}, alongside the number of peaks supplied to the embedding pipeline.
\begin{table}[h!]
  \begin{center}
    \caption{Chosen ChIP-seq experiments}
    \label{tab:table1}
    \begin{tabular}{l|c|r} % <-- Alignments: 1st column left, 2nd middle and 3rd right, with vertical lines in between
      \textbf{TF} & \textbf{Cell line} & \textbf{ENCODE accession}\\
      \hline
      MYC & GM12878 & ENCFF765CKK\\
      MYC & H1 & ENCFF700CXD\\
      MYC & K562 & ENCFF043WTJ\\
      CTCF & GM12878 & ENCFF017XLW\\
      CTCF & H1 & ENCFF692RPA\\
      CTCF & K562 & ENCFF769AUF\\
      SPI1 & GM12878 & ENCFF881ARS\\
      SPI1 & GM12878 & ENCFF198SPL\\
      SPI1 & K562 & ENCFF185DGL\\
    \end{tabular}
  \end{center}
\end{table}
Sequence is extracted from the human reference assembly hg38.
BED files are downloaded programmatically from the ENCODE portal.

\subsection{Reproducibility and seeding}

Stochastic elements of the pipeline are seeded explicitly to support reproducibility of intermediate quantities.
A fixed random seed of~42 is applied at every sampling and stochastic-fitting site, including peak subsampling, and UMAP fitting; this differs from the procedure of \citet{faltejskovaNonconsensusFlankingSequence2026a}, in which peak subsampling is performed without a fixed seed.
\emph{The full source code and configuration files used to produce the embedded set and the analyses reported in subsequent chapters are available at the project repository\footnote{\url{https://github.com/TypeTwo30312/tv-thesis}}.} % redundant

\section{Pipeline}

This section describes the deterministic chain from ENCODE BED files to per-peak embedding tensors stored on disk.
The three stages -- sequence extraction, embedding extraction, and the computational environment in which they run -- are each implemented as a separate Jupyter notebook in the project repository, and are reported here in the order in which they execute.

\subsection{Sequence extraction}
\label{sec:windows}

Window extraction reads an ENCODE narrowPeak file and outputs a per-peak record containing the peak coordinates and a fixed-length DNA sequence centred on the binding site.
The 10\,kb window length is chosen to match the $\pm 5000$\,bp range over which \citet{faltejskovaNonconsensusFlankingSequence2026a} report statistically significant compositional structure around in vivo binding sites.
The peak midpoint is used as the binding-site proxy: for the ENCODE narrowPeak files inspected in this thesis, the reported summit coincides with the midpoint of the peak interval (CTCF GM12878 peaks are uniformly $\sim$356\,bp wide, MYC GM12878 peaks $\sim$390\,bp, MYC H1 $\sim$264\,bp), and the midpoint convention matches that used by \citet{faltejskovaNonconsensusFlankingSequence2026a}.

Peaks whose 10\,kb window would extend past a chromosome boundary are dropped rather than padded, since padding with non-biological characters would introduce a fixed-position bias into any subsequent positional analysis.
Non-ACGT characters in the extracted sequence are masked to N at the extraction stage; the analysis-stage filtering of windows containing fully-N 50\,bp bins is described in (\ref{sec:nfilter}).
For each BED file, peaks are uniformly subsampled to a maximum of $10\,000$ (all peaks are retained if fewer are available), matching the \texttt{MAX\_SIZE\_SAMPLED} parameter used by \citet{faltejskovaNonconsensusFlankingSequence2026a} but with a fixed random seed of~42 throughout.
Each output row carries the columns \texttt{peak\_id}, \texttt{chr}, \texttt{start}, \texttt{end}, \texttt{center}, and \texttt{sequence}, and is written as a gzip-compressed CSV.

\subsection{Embedding extraction}
\label{sec:embeddings}

Embedding extraction is performed with the Evo~2 7B checkpoint released by \citet{brixiGenomeModellingDesign2026}; the architecture and pretraining objective are described in the background chapter (\ref{sec:evo2bg}).
The 7B family is used because it runs natively in bfloat16 on Ampere-class hardware and does not require Transformer Engine, FP8 support, or multiple Hopper-class GPUs which are needed for the 20B and 40B checkpoints.
Within the 7B family, the 1\,M-context variant (\texttt{evo2\_7b}) is selected over the 8\,kb-context base variant (\texttt{evo2\_7b\_base}) because the chosen 10\,kb window exceeds the latter's context length.
A single intermediate-layer embedding is extracted per token from the layer \texttt{blocks.28.mlp.l3}; intermediate layers are recommended over the final layer for representation learning because the final layer is specialised to the next-token prediction objective, and \texttt{blocks.28.mlp.l3} is the layer used by the exon-classifier example in the official Evo~2 repository \cite{brixiGenomeModellingDesign2026}.
Each 10\,kb sequence is tokenised under the Evo~2 byte-level scheme into a $(1, 10000)$ integer tensor (one token per nucleotide), passed through the model in a single forward pass on one GPU, and the resulting $(1, 10000, 4096)$ bfloat16 activation tensor is reshaped into 200 non-overlapping 50\,bp bins and averaged within each bin to yield a $(200, 4096)$ representation per peak, which is cast to fp16 before writing. A BOS (begginig of sequence) token, "0" in Evo2's tokenizer, is prepended to each sequence prior to embedding and is discarded before binning and saving. This is to mitigate the warmup artifact described in \ref{sec:positional}.
The 50\,bp bin width matches the $k$-mer resolution adopted by \citet{faltejskovaNonconsensusFlankingSequence2026a} so that per-bin readouts are directly comparable; non-overlapping bins are used in place of their 25\,bp stride because each bin already integrates 50 input positions of the same forward pass and stride overlap would not contribute additional information, while doubling computational requirements in downstream tasks.
Unlike the strictly local hand-engineered features used by \citet{faltejskovaNonconsensusFlankingSequence2026a}, each binned embedding has the full 10\,kb window in its causal receptive field via the autoregressive forward pass, and so reflects model-internal aggregation of context up to that point rather than a fixed-window feature of the sequence itself.
The output of this stage is, per BED file, a single \texttt{.npz} archive containing the $(n_\text{peaks}, 200, 4096)$ fp16 array together with a \texttt{.parquet} sidecar carrying the \texttt{peak\_id} column and BED-derived metadata in the same row order.
Embedding extraction was performed on a single NVIDIA A40 GPU (46\,GB VRAM) with peak allocation of 19.17\,GB at batch size~1, at an observed throughput of approximately 0.5 peaks per second; the embedded set comprises the 6 ChIP-seq files of \ref{sec:chip-data} totalling over 59\,GB. % update as i add data

\subsection{Computational environment}
\label{sec:env}

The embedding stage runs on the CERIT-SC JupyterHub platform under the NVIDIA PyTorch 2.5.0 container image, which provides a Python 3.10, CUDA 12.6, and Ubuntu 22.04 base; a project-specific Python 3.12 virtual environment is layered on top, installed into the persistent home directory and pinning \texttt{torch}~2.7.1+cu128, \texttt{flash-attn}~2.8.0.post2, and \texttt{evo2}.

A separate CPU-only environment is used for the analyses reported in \ref{sec:methods-analyses}, isolating the heavy GPU stack from sessions that do not require it and pinning \texttt{numpy}, \texttt{polars}, \texttt{scikit-learn}, and \texttt{umap-learn} alongside the standard scientific Python packages.
The full code base, configuration files, and notebooks are available at the project repository\footnote{\url{https://github.com/TypeTwo30312/tv-thesis}}.

\section{Analysis}

\subsection{Global mean-pool PCA}

For each peak, the $(200 \times 4096)$ activation tensor was averaged over the positional axis, yielding a 4096-dimensional mean-pool vector.
Vectors from all files were stacked into a single array ($n = 49{,}497$ peaks after N-bin filtering) and subjected to PCA (50 components) and UMAP ($n_\text{neighbors} = 30$, $\text{min\_dist} = 0.1$).
Per-peak mean GC content was calculated as the mean of the per-bin GC fractions across all 200 bins.
Pearson correlation between PC scores and mean GC content was calculated post hoc.
One-way ANOVA was used to estimate the proportion of variance in PC scores explained by transcription factor identity and BED file identity ($\eta^2$ = between-group sum of squares / total sum of squares).

\subsection{Positional profiles}
\label{sec:positional}

File-level mean positional profiles were obtained by averaging each file's per-peak activation tensors across peaks, yielding one $(200 \times 4096)$ profile per file.
The six profiles were stacked into a $(1200 \times 4096)$ array.

The Evo~2 model conditions each token's representation on preceding tokens only.
Tokens near the left window edge therefore have little or no left context, producing anomalous activations in the first bins of each profile.
Inclusion of the BOS token during embedding (Section~\ref{sec:embeddings}) partially mitigates this effect, but does not eliminate it.
To prevent this artefact from dominating the fit, the first 15 bins (750~bp) were excluded from PCA fitting; all 200 bins were subsequently projected onto the fitted components for visualisation, with the trim boundary marked.

Two sequential analyses were performed on the stacked profiles.

\textbf{Raw positional PCA (Block~2).}
The stacked array was standardised column-wise using \texttt{StandardScaler} \cite{pedregosaScikitlearnMachineLearning2011} (zero mean, unit variance per embedding dimension across all rows) and subjected to PCA (20 components, \texttt{random\_state}~=~42).
Per-bin embedding magnitude was computed separately as the Euclidean norm of each row of the per-file profile and used as a diagnostic reference for the warm-up region.

\textbf{L2-normalised positional PCA (Block~2b).}
To isolate directional variation in the file-mean profiles independently of overall activation magnitude, each row of the stacked file-mean array was scaled to unit L2 norm prior to PCA.
Each row corresponds to one (file, bin) pair; dividing by its Euclidean norm projects it onto the unit sphere in the embedding space, so that only the direction, not the magnitude, of the mean activation at that position contributes to subsequent components.
The warm-up trim was applied before normalisation.
Column-wise standardisation via \texttt{StandardScaler} was then applied to the L2-normalised array, followed by PCA (10 components, \texttt{random\_state}~=~42).

Per-bin mean GC content was calculated from the extracted sequences as the fraction of G and C bases per 50~bp bin, averaged across peaks within each file.
Dinucleotide frequencies were calculated analogously, as the frequency of each dinucleotide pair within each 50~bp bin, averaged across peaks.
Pearson correlations between PC scores and per-bin GC content, embedding magnitude, and the directional dinucleotide contrasts reported by \citet{faltejskovaNonconsensusFlankingSequence2026a} (AA$-$TT, CA$-$TG, AC$-$GT) were computed post hoc across trimmed bins for each file.


```