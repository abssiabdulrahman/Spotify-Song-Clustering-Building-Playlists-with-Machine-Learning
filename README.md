# Spotify Song Clustering — Building Playlists with Machine Learning

A data science project that turns 5,235 Spotify songs into 41 musically coherent playlists using unsupervised machine learning.

## The brief

A small-scale music startup called **Moosic** currently hand-curates all its playlists. They want to know whether machine learning can scale this work without sacrificing quality.

The hard constraint: every playlist must contain between **50 and 250 songs**.

My job was to build a prototype, evaluate it honestly, and recommend whether ML is a viable path forward.

## TL;DR — the final result

| Metric | Value |
|---|---|
| Songs in delivery | 5,230 |
| Final playlists | 41 |
| Playlist size range | 50–250 (fully compliant) |
| Mean per-song silhouette | 0.14 |
| Songs correctly placed | 88% |
| Strong + Good playlists | 19 |
| Incoherent playlists | 0 |

**My recommendation to Moosic: don't replace the curators, augment them.** ML groups thousands of songs in seconds; experts spend a day refining the output instead of weeks building from scratch.

## Repository contents

```
.
├── README.md                          ← you are here
├── spotify_clustering_final.ipynb     ← the full analysis notebook
├── moosic_presentation.pptx           ← 5-minute stakeholder pitch

```

## The approach in 4 steps

1. **Clean the data** — drop only the 5 rows with impossible values per Spotify's API spec (tempo=0 or loudness>0). Keep all real outliers because they represent genuine musical diversity (classical, ambient, rap, live recordings).
2. **Scale with StandardScaler** — feature ranges span 7 orders of magnitude, so unscaled K-Means would be dominated by `duration_ms`.
3. **K-Means with a split-and-merge post-processing pipeline** — K-Means alone cannot satisfy the 50–250 size brief at any k. My custom pipeline recursively splits oversized clusters and merges undersized ones until full compliance.
4. **Sub-cluster the leftover bucket** — one cluster of ~115 songs came out incoherent. Instead of dropping those songs, I ran K-Means again on just that subset to split them into smaller, tighter sub-groups.

## Key decisions and why I made them

| Decision | Choice | Reasoning |
|---|---|---|
| Outlier handling | Keep all real outliers | They are real musical diversity, not data errors. Removing them collapses musical range. |
| Error handling | Drop 5 rows | These violate Spotify's documented value ranges, so they are clearly bugs. |
| Scaling | StandardScaler | Distance-based algorithm, real outliers worth preserving, no normality assumption needed. |
| Feature selection | 9 features (drop `duration_ms`) | Tested empirically — dropping duration improved mean silhouette from 0.108 to 0.136 and cut misplacement nearly in half. |
| Number of clusters | Starting k = 39 | Tested every starting k from 21 to 50. k=39 produced the fewest incoherent clusters with strong silhouette. |
| Size compliance | Split-and-merge pipeline | K-Means has no native size constraint, so I built one. |
| Leftover handling | Sub-cluster, do not drop | Tested both approaches. Sub-clustering preserved all 5,230 songs with the same coherence improvement as dropping. |

## How I measured whether playlists were actually cohesive

I used **silhouette scoring**. For every single song:

- Compute `a` = average distance from this song to other songs in its own playlist
- Compute `b` = average distance from this song to songs in the nearest other playlist
- Silhouette score = `(b - a) / max(a, b)`

This produces a number between -1 and +1 for every song:
- Positive → song is correctly placed
- Negative → song would fit better in a different playlist

I then averaged the per-song scores within each playlist to classify them:

| Tier | Threshold | Meaning |
|---|---|---|
| STRONG | > 0.25 | Songs clearly belong together |
| GOOD | 0.15 – 0.25 | Reasonable coherence |
| OK | 0.05 – 0.15 | Acceptable but fuzzy edges |
| WEAK | 0 – 0.05 | Marginal |
| INCOHERENT | < 0 | Songs fit better elsewhere |

Final breakdown across my 41 playlists: **2 Strong, 17 Good, 19 OK, 3 Weak, 0 Incoherent**.

## What I learned

- **Silhouette score has structural bias toward fewer clusters.** The mathematical "winner" was k=21, but that sat at the boundary of the size limit and made the playlists risky.
- **Outliers in music data are usually real signal, not noise.** Inspecting the actual extreme songs (classical symphonies, ambient pieces, rap tracks) showed they were the most distinctive content in the catalogue.
- **K-Means cannot enforce business rules natively.** A simple split-and-merge post-processing step solves this cleanly without specialised libraries.
- **Feature selection matters even with continuous numerical data.** Dropping `duration_ms` halved the misplacement rate. Two songs of the same length are not musically similar.
- **Sub-clustering beats dropping.** Same coherence improvement, no data loss.
- **Audio features alone are not enough to fully match human curators.** They capture sonic qualities (energy, mood, tempo) but miss language, genre conventions, cultural context, and listener behaviour.

## Methods used

- **Algorithm:** K-Means (scikit-learn) with `k-means++` initialisation and `n_init=10` for stability
- **Scaling:** StandardScaler
- **Evaluation:** silhouette score (global and per-song via `silhouette_samples`)
- **Post-processing:** custom recursive split + nearest-neighbour merge
- **Stack:** pandas, numpy, scikit-learn, matplotlib, seaborn

## What I would do next

In order of impact:

1. **Add metadata features** — genre tags, artist, release year. Available via Spotify API. Would meaningfully improve cluster coherence.
2. **Hybrid workflow with curators** — let experts refine the 41 algorithm-generated playlists in a day rather than build from scratch over weeks.
3. **Capture user listening data** — once there are real users, what songs they play together is the strongest signal for what belongs together.
4. **Benchmark other algorithms** — constrained K-Means (`k-means-constrained`), hierarchical clustering.
5. **A/B test against curators** — launch 20 ML playlists alongside 20 human-curated ones and measure user engagement.



## About me

Made by (www.linkedin.com/in/
abdul-rahman-abssi-b64bb640b)
