# GeoResearchPlatformPublic

## Neighbourhood Feature Catalogue

MineTRACE scores a location from the geochemistry in its neighbourhood. For each query point, the feature extractor gathers every assay sample within **5, 10, and 50 km** and summarises them into the features listed below. All element-level statistics are computed on `log(1+x)`-transformed concentrations to damp the heavy right-skew and below-detection censoring typical of assay data, and each statistic is instantiated for every element in a fixed **14-element panel** of target metals and common pathfinders. These features are the raw evidence consumed by the interpretable expert tree (see the paper, *Interpretable Prospectivity Engine*); this table is a reference for anyone reproducing or auditing the scorer.

A statistic is only emitted where at least **three valid samples** fall inside the radius. Sparsely sampled locations therefore produce nulls, and the geochemistry abstains rather than extrapolating.

| Feature | Description |
|---|---|
| `{el}_{s}km_median` | Median log-concentration (`log(1+x)`) of element *el* among samples within *s* km. |
| `{el}_{s}km_p90` | 90th percentile of log-concentration within *s* km. |
| `{el}_{s}km_cv` | Coefficient of variation of log-concentration within *s* km. |
| `{el}_{s}km_frac_above` | Fraction of samples within *s* km whose log-concentration exceeds the element's regional 75th-percentile threshold. |
| `{el}_contrast_5_50`, `{el}_contrast_10_50` | Local-to-regional median contrast in log space (log-median at 5 / 10 km minus log-median at 50 km), measuring enrichment over regional background. |
| `ratio_{A}_{B}` | Log-ratio of elements *A* and *B* at 10 km (difference of their log-medians), capturing pathfinder and element-association signals. |
| `spatial_coherence_10km` | Mean pairwise distance among the five highest-value samples (ranked by the panel's primary element) within 10 km; smaller values indicate a coherent anomaly rather than isolated spikes. |
| `PC{i}_{s}km_median` | Median of the *i*-th multi-element principal-component score within *s* km (*s* ∈ {5, 10}, *i* = 1…5). Defined by the extractor but disabled in the deployed scorer (`with_pca=False`), so these are null by default. |
| `n_samples_{s}km` | Number of samples within *s* km — local data density and coverage. |

**Radii.** `s ∈ {5, 10, 50}` km.

**Element panel (14).** Cu, Au, Mo, Ag, Pb, Zn, Co, Bi, Ni, W, As, Sn, Cr, Sb.

**Candidate ratios.** Instantiated whenever both elements are present in the panel: Cu/Mo, Cu/Pb, Au/Cu, Zn/Pb, Co/Ni, W/Sn, W/Mo, Au/As, Ni/Co, Ni/Cr, Sb/As, Bi/Sb.

**Notes.**
- Element-level concentration statistics (`median`, `p90`, `cv`, `frac_above`, `contrast`, `ratio`) are computed on `log(1+x)`. `n_samples` (a count), `spatial_coherence` (a distance), and the PC medians are *not* log-concentration statistics.
- The `frac_above` threshold is the element's regional 75th percentile of `log(1+x)`, estimated over the full sampling source.
- Every element-level row is instantiated for each of the 14 panel elements; target metals outside the panel (e.g. Ta, Mn) have no element-level features for the target element itself and are scored through their pathfinders and ratios.
