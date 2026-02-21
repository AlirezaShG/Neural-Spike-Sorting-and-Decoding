# Report

English translation of the original project report (19 pages, Persian). The
text is the author's, with the section structure preserved. Figure numbers
refer to the original document.

The report was written against the original scripts. Where a defect found
while rebuilding the notebooks changes what a figure actually shows, a
*Correction* note follows the paragraph — the original text is left intact
rather than quietly edited, since the analysis history is part of the point.

---

## 1 — Spike sorting from scratch

**Figure 1 — Voltage distribution.** First we plot the voltage
distribution. Viewed in microvolts, it is a normal distribution centred on
zero.

**Figure 2 — Raw signal and after filtering.** After filtering the data we
see that the amplitude of the signal is somewhat reduced. Filtering serves
both to remove noise and to find the spikes, because spikes are sudden,
fast (high-frequency), low-amplitude deflections in the signal.

**Figure 3 — All detected spikes as waveforms.** The peaks we found were
cut 2 ms before and after, turned into waveforms, and aligned at the
origin.

> **Correction.** The detector searched for positive-going peaks only.
> Extracellular action potentials in this recording are predominantly
> negative-going: detecting both polarities takes the count from 8,999 to
> 31,064 events, so roughly 71% of the spikes were being discarded. The
> notebook detects both.

**Figure 4 — Feature extraction from the detected waveforms (PCA).** Given
the waveform of each spike we can separate them into distinct clusters with
PCA. These clusters correspond to neurons. From the figure it appears we
found only one neuron; the remaining points, scattered and far from the
main cluster, can be treated as noise.

**Figure 5 — PCA with different values of K.**

### Comparison with `spikes.mat`

To compare against the indices in `spikes.mat`, we first downsampled the
original signal at several rates. The best match was at rate 30, i.e. we
converted the signal to 1000 Hz (from 30 kHz). We then compared the spikes
found in our own data, with a 1 ms tolerance, against the spikes in
`spikes.mat`. About 1000 matched, which is 1% of the spikes in
`spikes.mat`; the F1 score was 0.01. It appears that with downsampling, two
clusters — that is, two neurons — were found.

> **Correction — this comparison does not measure detection accuracy.** The
> reference file holds one array, `ind_spikes_it`, of 90,789 strictly
> increasing unique integers. 73.7% of its consecutive differences are
> exactly 1, it covers 65.4% of the range 1…138,879, and it ends exactly at
> 138,879. A spike train over this 2,390 s recording sampled at 1 kHz would
> span about 2.39 million samples and could not have one-sample gaps three
> quarters of the time. It is a 1-based selection index into a
> 138,879-entry list that is not in the file.
>
> The consequence: every one of the 1,004 "matches" falls in the first
> 138.9 s of a 2,390 s recording. Of the 1,385 detections that land in that
> range, 1,004 (72.5%) "match" — and the reference covers 65.4% of its own
> range, so a random integer scores about the same. The F1 of 0.018
> describes integer collisions, not detection performance.
>
> The notebook keeps the calculation because the report quotes it, and adds
> the diagnostics above beside it. In place of it, cluster quality is
> measured the way it can be measured without ground truth: silhouette
> separation in PCA space, refractory-period (ISI < 2 ms) violation rate per
> cluster, and amplitude SNR. At k = 3 the silhouette is 0.525, with
> violation rates of 5.8% and 6.2% for the two large clusters — plausible
> single units — and 49.4% for a third cluster of 86 events, which is not a
> unit.

**Figure 6 — PCA and t-SNE for the newly detected spikes.** The axes of a
t-SNE plot are essentially meaningless; it matters only for visualisation
and for preserving cluster structure.

**Figure 7 — t-SNE before downsampling.** More clusters can be found in
this t-SNE. Also, with the threshold `0.9 × max` we find only one spike in
total, for which neither PCA nor t-SNE can be drawn.

> **Correction.** This observation is correct and is worth stating as a
> general point: `0.9 × max|x|` is not a threshold choice, it is a way of
> selecting the single largest sample in the recording — nothing short of
> the extremum clears 90% of it. In the rebuilt notebook the arm is kept
> (the report quotes its F1) and the embedding is skipped with a message,
> but the comparison the section is reaching for is liberal versus
> conservative *in units of the noise*, so an 8σ arm is added alongside the
> 4σ one.

### Manual sorting with ROSS

**Figure 8 — Denoising for cluster 1.** First all clusters are merged into
one large cluster, then the denoising step is run.

**Figure 9 — After manual sorting.**

**Figure 10 — After a second resort.** Clicking on the main cluster and
resorting again splits it into 7 clusters.

**Figure 11 — Waveform shapes of the recovered clusters.** The waveform
shapes differ between clusters. One could be stricter still and split these
clusters further, extracting more clusters and more distinct waveform
shapes.

**Figure 12 — PCA1 and PCA2 against time.**

---

## 2 — Single-neuron activity

### PSTH

**Figure 13 — PSTH of two selected neurons.** Most neurons were more
sensitive to the face images, but there were also neurons whose responses
were scattered and not interpretable. For the PSTH — and for the rest of
the analyses here — we used a sliding time window and counted the spikes
falling inside each window, producing PSTH curves with overlapping windows,
which were then used as features for the SVM.

### Fano factor

**Figure 14 — Fano factor.** The Fano factor measures how random a neuron's
behaviour is: the lower it is, the further the response is from a random
(Poisson) process. As the figure shows, in the baseline period the neuron
behaves entirely randomly, and immediately after onset purposeful,
stimulus-driven behaviour appears — which agrees with both the hypothesis
and the observations. However, given that our neurons are more sensitive to
the face class, we would have expected this measure to be lower for face
than for the others. The smoothing applied to the curve is the likely
reason it is not visible; the expected overall shape is nevertheless
obtained.

> **Note.** The original pooled trials across neurons using neuron 0's
> stimulus order, while the decoding and mutual-information cells resolved
> the trial index against each neuron's own stimulus list. Both cannot be
> right. The rebuilt notebooks use the per-neuron lookup everywhere, so the
> counts entering one variance are guaranteed to come from the same image.

### SVM decoding

**Figure 15 — SVM results.** The results were entirely natural and
predictable. Taking the image onset as time 100, it is clear that neural
activity increases after the image is shown, and it follows that SVM
accuracy peaks after the neurons respond, because before and after that
window we are in baseline and there is no particular information. From the
recall curve one can also conclude that our neurons are mostly face-related,
and that this class separates strongly from the others.

> **Correction.** The chance line was drawn at 25%. The four categories
> contain 200, 120, 70 and 110 images, so a classifier that always answers
> *face* already scores 40%. The notebook draws the majority-class rate.
> Separately, the original standardised each time slice before
> cross-validating it, so every fold's held-out rows had contributed to the
> transform; the scaler is now fitted inside each fold.

### Time–time decoding

**Figure 16 — Temporal generalisation.** On the main diagonal (test time ==
train time) the model is trained and tested at the same latency, and this
is where the highest accuracy is seen, above 65% — meaning that in those
windows the neural responses support the best classification. The width of
the diagonal reflects the stability of the responses: training at one
latency carries useful information for later ones. For instance, a model
trained at 250 ms also classifies well over a nearby range such as
200–300 ms.

Meaningful accuracy begins around 100 ms after stimulus presentation, which
is the initial response latency of the temporal region to stimuli. Several
distinct clusters appear in the 150–200 ms range, showing that the
categories are separable by the population across several time windows and
that category information is preserved through this interval.

> **Correction — every latency in this figure is 100 ms early.** The PSTH
> bin offsets were counted from the start of the *cropped* epoch, which
> begins at 100 ms, and that offset was never added back. The figure
> labelled 100–320 ms actually shows 200–420 ms. Onset of decodable
> information is therefore about 200 ms, not 100 ms — which is what the
> mutual-information section below independently reports, and what the
> decoding curve in the SVM section shows. The corrected axis brings the
> three into agreement.

### Mutual information over time

**Figure 17 — Mutual information.** The window in which mutual information
is greatest is roughly 200–300 ms after the stimulus, indicating that
category information is highest there. This agrees with the decoding and
Fano-factor analyses: all three methods place the greatest separation
between categories in the same window, and the Fano-factor change reflects
the shift in response variability over the same interval. Taken together,
these findings indicate that in inferior temporal cortex the window
200–300 ms after stimulus presentation is the most critical period for
processing and separating category-related visual information.

> **Correction.** The permutation p-values were estimated as
> `mean(null >= observed)`, which returns exactly 0 when no permutation
> beats the observation and was then compared against a Bonferroni
> threshold of 0.05/51 ≈ 9.8 × 10⁻⁴. With 1000 permutations the finest
> resolvable p-value is 10⁻³ — the same order as the threshold — so
> "significant" and "we ran out of permutations" could not be told apart.
> The notebook uses the add-one estimator and notes that resolving a
> corrected effect needs permutations in the tens of thousands.

### Category discriminability with d-prime

**Figure 18 — d-prime.** Again the results were predictable. The face class
clearly differs most from the rest; we recorded the most spikes for face,
and this was expected. Classes that resemble each other differ less from
each other. And because body falls in the same broad class as face
(animate), it differs sharply from the inanimate classes.

> **Correction.** The y-axis was labelled "d′ (bits)". d-prime is a
> standardised mean difference and is dimensionless; bits belong to the
> mutual-information section.

---

## 3 — Population activity

### Representational dissimilarity and Kendall's tau

**Figure 19 — RDM.**
**Figure 20 — Ground-truth (category model) RDM.**

**Figure 21 — Kendall's tau against the RDM.** The Kendall curve agrees
fully with the earlier analyses: the greatest similarity between our RDM
and the ground-truth matrix occurs at the same latency where mutual
information and SVM accuracy peak. Also, because our neurons are more
sensitive to faces, the face class stands apart from the others in the RDM,
as though we really had two classes — face, and everything else. This was
entirely predictable, since these neurons were recorded from temporal
cortex, where most face-processing activity takes place.

> **Note.** The single-latency RDM (Figure 19) was taken at a hardcoded bin
> index that corresponds to 75 ms — before category information is
> decodable, by this report's own account. Its near-absence of block
> structure is therefore the expected result rather than a null finding.
> The notebook states the latency explicitly for that reason. The grid of
> RDMs across the response window was titled "150–300 ms in 10 ms bins" but
> selected bins on a 5 ms grid and then silently truncated to the 16
> available panels, so it actually showed 150–225 ms; the notebook uses the
> 10 ms grid, which gives exactly 16 bins covering the stated window.

---

## 4 — Phase–amplitude coupling and spectral analysis

**Figure 22 — Two sessions compared, face class.** If these two sessions
were recorded from different points in the brain, the differences in their
PAC patterns may reflect regional characteristics and the differing
functions of those areas. Each region may have its own oscillatory patterns
reflecting its particular processing mechanisms. For example, the region
recorded in session S0 may be more involved in phase processing in the
theta-to-alpha bands with amplitude in gamma, while the region recorded in
session S1 may show amplitude patterns in the beta-to-low-gamma bands and a
different phase frequency.

These differences matter because they help us understand how different
regions process information through the coordination of slow and fast
oscillations, and how the interaction between those frequencies is
organised.

**Figure 23 — Mean PAC for the face class.** The MI and Canolty analyses
differ substantially, because MI measures the deviation of the
amplitude-by-phase distribution from uniform (scaled, and independent of
power), whereas Canolty measures the mean of amplitude × phase and is
therefore dependent on amplitude power.

**Figure 24 — Mean PAC for the body class.** As the figure shows, the
frequencies at which coupling occurs differ across categories. Under the
Canolty measure, the body category shows coupling between a low phase
frequency in theta (around 6 Hz) and higher amplitude frequencies in beta
and gamma (around 40 Hz), while in the face category the same coupling
occurs at lower frequencies, in the alpha and theta bands.

**Figure 25 — Mean PAC for the artificial class.** This heatmap is the mean
PAC under the Canolty measure for the artificial category in the first of
17 sessions. The strongest coupling appears at an amplitude frequency of
roughly 30–50 Hz, indicating coupling of a low phase frequency (about
5–8 Hz) with amplitude in low gamma. Coupling strength in this category is
weaker and more scattered than in the others, which may be due to noise or
unstructured activity in these signals. The phase frequency lies in theta
and low alpha, the range that usually matters for the coordination of
neural oscillations. Overall the pattern shows that phase–amplitude
coupling in the artificial category concentrates in the lower amplitude
bands with theta-to-alpha phase, but at lower strength and with greater
dispersion than the categories corresponding to real stimuli.

**Figure 26 — Difference heatmap, body minus face.**

**Figures 27, 28, 29 — PAC across the remaining classes.** There are clear
differences in PAC pattern between categories. In the body category,
coupling between theta phase (around 6 Hz) and beta–gamma amplitude (around
40 Hz) is stronger, whereas in the face category coupling appears more at
lower amplitude frequencies (alpha or below). The artificial category has a
different pattern again, weaker and more scattered, probably because of the
noisy or unstructured nature of that data; natural behaves similarly to it
while still showing a distinct pattern. These differences indicate that the
brain adopts distinct oscillatory patterns when processing each stimulus
category, and that PAC can serve as a marker of those neuro-sensory
differences.

> **Correction.** All of these comodulograms were computed on the
> trial-averaged LFP. Averaging across trials retains only the part of the
> signal that is phase-locked to stimulus onset and cancels the rest; gamma
> amplitude is largely *induced* rather than phase-locked, so trial
> averaging removes most of what the analysis is measuring, and what
> survives is dominated by the evoked potential. The comodulogram of the
> average is not the average comodulogram. The notebook computes PAC per
> trial and averages the resulting maps.
>
> Two further points. Neither estimator was surrogate-corrected: raw MI and
> MVL are both biased upward by signal length, filter bandwidth and
> non-sinusoidal waveform shape, so an uncorrected comodulogram has no zero
> point against which "coupling" can be judged. The notebook offers
> surrogate z-scoring behind a flag. And the per-session results were
> accumulated across all 17 sessions and then never plotted — the
> grand-average comodulogram, which is where a consistent coupling peak
> would show up, is added.

### Power spectrum

**Figure 30 — Power spectrum for body and face.**
**Figure 31 — Power spectrum for artificial and natural.**

As the earlier images and results make clear, the body and face classes —
the animate stimuli — show the highest signal power at low frequencies, a
consequence of broader neural processing and greater information exchange.
This may follow from the greater salience of these stimuli to the nervous
system and the more complex processing they require. The natural and
artificial classes, by contrast, are noisier by virtue of being inanimate,
and show lower power in the low-frequency range (the theta and alpha
bands). These differences reflect the different ways the brain's
oscillatory activity responds to different kinds of stimuli.
