<div align="center">

# 🎵 Spotify Song Clustering
### Building Playlists with Machine Learning

*Turning 5,235 Spotify songs into 41 musically coherent playlists with unsupervised ML*

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)

</div>

---

## 📋 The Brief

> A small-scale music startup called **Moosic** currently hand-curates all its playlists. They want to know whether machine learning can scale this work — without sacrificing quality.

**The hard constraint:** every playlist must contain between **50 and 250 songs**.

My job: build a prototype, evaluate it honestly, and recommend whether ML is a viable path forward.

---

## 🎯 TL;DR — Final Result

<div align="center">

| 📊 Metric | 🎯 Value |
|:---|:---:|
| 🎵 Songs in delivery | **5,230** |
| 📂 Final playlists | **41** |
| 📏 Playlist size range | **50–250** ✅ |
| 📈 Mean per-song silhouette | **0.14** |
| ✅ Songs correctly placed | **88%** |
| ⭐ Strong + Good playlists | **19** |
| ❌ Incoherent playlists | **0** |

</div>

> 💡 **My recommendation to Moosic:** *Don't replace the curators — augment them.*
> ML groups thousands of songs in seconds; experts spend a day refining the output instead of weeks building from scratch.

---

## 📁 Repository Contents

```
📦 spotify-clustering
 ┣ 📄 README.md                          ← you are here
 ┣ 📓 spotify_clustering_final.ipynb     ← the full analysis notebook
 ┗ 🎞️  moosic_presentation.pptx           ← 5-minute stakeholder pitch
```

---

## 🛠️ The Approach in 4 Steps

```mermaid
graph LR
    A[🎵 5,235 raw songs] --> B[🧹 Clean: drop 5 errors]
    B --> C[📐 Scale with StandardScaler]
    C --> D[🔀 K-Means + split-merge]
    D --> E[🎯 Sub-cluster leftovers]
    E --> F[✅ 41 playlists]
```

**1️⃣ Clean the data** — Drop only the 5 rows with impossible values per Spotify's API spec (`tempo=0` or `loudness>0`). Keep all real outliers because they represent genuine musical diversity (classical, ambient, rap, live recordings).

**2️⃣ Scale with StandardScaler** — Feature ranges span 7 orders of magnitude. Without scaling, K-Means would be dominated by `duration_ms`.

**3️⃣ K-Means with a split-and-merge pipeline** — K-Means alone cannot satisfy the 50–250 size brief at any k. My custom pipeline recursively splits oversized clusters and merges undersized ones until full compliance.

**4️⃣ Sub-cluster the leftover bucket** — One cluster of ~115 songs came out incoherent. Instead of dropping those songs, I ran K-Means again on just that subset to split them into smaller, tighter sub-groups.

---

## 🧠 Key Decisions

| 🔧 Decision | ✅ Choice | 💭 Reasoning |
|:---|:---|:---|
| **Outlier handling** | Keep all real outliers | They are real musical diversity, not data errors |
| **Error handling** | Drop 5 rows | These violate Spotify's documented value ranges |
| **Scaling** | StandardScaler | Distance-based algorithm, preserves real outliers |
| **Feature selection** | 9 features *(drop `duration_ms`)* | Improved mean silhouette **0.108 → 0.136** |
| **Number of clusters** | Starting k = 39 | Tested k=21 to 50; k=39 minimised incoherent clusters |
| **Size compliance** | Split-and-merge pipeline | K-Means has no native size constraint, so I built one |
| **Leftover handling** | Sub-cluster, do not drop | Same coherence win as dropping, zero data loss |

---

## 📐 How I Measured Whether Playlists Were Cohesive

I used **silhouette scoring**. For every single song:

```
        b - a
sil = ─────────
       max(a, b)
```

> `a` = average distance from this song to other songs in **its own** playlist
> `b` = average distance from this song to songs in the **nearest other** playlist

This produces a number between **-1** and **+1** for every song:
- 🟢 **Positive** → song is correctly placed
- 🔴 **Negative** → song would fit better in a different playlist

I then averaged the per-song scores within each playlist:

<div align="center">

| 🏆 Tier | 📊 Threshold | 🎵 Meaning |
|:---:|:---:|:---|
| 🟢 **STRONG** | `> 0.25` | Songs clearly belong together |
| 🟢 **GOOD** | `0.15 – 0.25` | Reasonable coherence |
| 🟡 **OK** | `0.05 – 0.15` | Acceptable but fuzzy edges |
| 🟠 **WEAK** | `0 – 0.05` | Marginal |
| 🔴 **INCOHERENT** | `< 0` | Songs fit better elsewhere |

</div>

**Final breakdown across my 41 playlists:**

```
🟢 STRONG       ██                  2
🟢 GOOD         █████████████████  17
🟡 OK           ███████████████████ 19
🟠 WEAK         ███                 3
🔴 INCOHERENT                       0  ✨
```

---

## 💡 What I Learned

> **Silhouette score has structural bias toward fewer clusters.**
> The mathematical "winner" was k=21, but that sat at the boundary of the size limit and made the playlists risky.

> **Outliers in music data are usually real signal, not noise.**
> Inspecting the actual extreme songs (classical symphonies, ambient pieces, rap tracks) showed they were the most distinctive content in the catalogue.

> **K-Means cannot enforce business rules natively.**
> A simple split-and-merge post-processing step solves this cleanly without specialised libraries.

> **Feature selection matters even with continuous numerical data.**
> Dropping `duration_ms` halved the misplacement rate. Two songs of the same length are not musically similar.

> **Sub-clustering beats dropping.**
> Same coherence improvement, no data loss.

> **Audio features alone are not enough to fully match human curators.**
> They capture sonic qualities (energy, mood, tempo) but miss language, genre conventions, cultural context, and listener behaviour.

---

## 🧪 Methods Used

| 🛠️ Component | 📦 Tool |
|:---|:---|
| **Algorithm** | K-Means (`k-means++` init, `n_init=10`) |
| **Scaling** | StandardScaler |
| **Evaluation** | `silhouette_score`, `silhouette_samples` |
| **Post-processing** | Custom recursive split + nearest-neighbour merge |
| **Stack** | `pandas` · `numpy` · `scikit-learn` · `matplotlib` · `seaborn` |

---

## 🚀 What I Would Do Next

*In order of impact:*

`🥇` **Add metadata features** — Genre tags, artist, release year via Spotify API. Would meaningfully improve cluster coherence.

`🥈` **Hybrid workflow with curators** — Let experts refine the 41 algorithm-generated playlists in a day, not weeks.

`🥉` **Capture user listening data** — What songs users play together is the strongest signal for what belongs together.

`4️⃣` **Benchmark other algorithms** — Constrained K-Means, hierarchical clustering.

`5️⃣` **A/B test against curators** — 20 ML playlists vs 20 human-curated ones, measure engagement.

---

<div align="center">

## 👋 About Me

**Abdul Rahman Abssi**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/abdul-rahman-abssi-b64bb640b)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/abssiabdulrahman)

</div>
