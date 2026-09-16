# Salmonella Typhimurium: Clinical vs. Environmental Classification from Gene Presence/Absence

A small ML project classifying *Salmonella enterica* serovar Typhimurium isolates as clinical or environmental based on AMR, stress-resistance, and virulence gene presence/absence, using data from NCBI's Pathogen Detection Isolates Browser (AMRFinderPlus calls). Built as a learning exercise, loosely inspired by genomic-AMR-prediction studies like Benefo et al. (2024).

## Data
2,889 Typhimurium isolates (1,708 clinical, 1,181 environmental), after:
- restricting to isolates whose NCBI-computed genomic serotype is exactly "Typhimurium" (excludes monophasic variants and genuine cross-serovar mismatches)
- dropping labeled vaccine strains
- verifying every "clinical" label against the isolate's Host field (dropped 21 isolates labeled clinical but sourced from non-human/undocumented hosts)

Features: 260 binary gene-presence columns built from the AMR/Stress/Virulence genotype calls, keeping only confident qualifiers (COMPLETE/POINT/PARTIAL_END_OF_CONTIG).

## Method
Random Forest as the primary model, compared against Logistic Regression, Gradient Boosting, Decision Tree, KNN, and Bernoulli Naive Bayes. All models evaluated with **GroupKFold cross-validation grouped by SNP cluster**, so no near-identical genome can appear in both train and test. A naive random split was checked first and found to leak 81.7% of test isolates this way, so it was discarded in favor of the grouped evaluation below.

## Results

| Check | AUROC |
|---|---|
| Naive random split (leakage, not trusted) | 0.902 |
| Random Forest, honest (GroupKFold) | 0.823 ± 0.066 |
| Logistic Regression | 0.851 ± 0.067 |
| Gradient Boosting | 0.843 ± 0.053 |
| K-Nearest Neighbors | 0.806 ± 0.050 |
| Decision Tree | 0.796 ± 0.053 |
| Bernoulli Naive Bayes | 0.777 ± 0.030 |
| Gene count only (no gene identity) | 0.657 ± 0.049 |
| Shuffled labels (null check) | 0.484 ± 0.019 |

All classifiers land well above the gene-count-only baseline and the shuffled-label null, so there's a genuine, model-agnostic, non-leaked discriminatory signal — not just an artifact of one algorithm or of assembly quality.

The strongest driver is `oafA` (O-antigen acetyltransferase), more common in clinical isolates. A mercury-resistance/plasmid signature (`merR`, `sul2`, `tet(A)`) leans environmental. Most top genes are broadly distributed across 300+ independent SNP clusters (a real recurring pattern, not one clone's fingerprint). Two exceptions, `qnrB19` and `ybtP`, are concentrated in a handful of clusters and likely reflect specific circulating lineages rather than a universal trait.

## Repo contents
- `Salmo_Typhimurium.ipynb` — full analysis notebook, annotated
- `fig1`–`fig4` — feature importance/direction, validation summary, ROC curve, clonal-concentration check
