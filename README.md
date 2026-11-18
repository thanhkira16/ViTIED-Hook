# ViTIED-Hook

This repository contains the release tables for the study **“The First Five Seconds: Assessing Multimodal Hook Features for Engagement Score Prediction in TikTok Videos.”** The dataset extends ViTIED with structured annotations and features from the first five seconds of Vietnamese TikTok videos.

## Files

| File | Videos | Creators | Role |
|---|---:|---:|---|
| `train.csv` | 5,711 | 286 | Training partition for the canonical seed-42 split |
| `stream.csv` | 1,335 | 72 | Creator-disjoint held-out validation partition for seed 42 |

Together the files contain 7,046 unique videos from 358 creators. There is no creator overlap between the two partitions. Here, `stream.csv` is the seed-42 validation partition; it is not a separate live-stream dataset.

## Dataset construction

The source snapshot contained 9,114 unique video IDs. Following the manuscript’s filtering procedure, inaccessible videos and videos outside the study scope were removed, leaving 7,046 videos. The final seed-42 split was generated with an 80/20 `GroupShuffleSplit` grouped by creator, using `random_state=42`.

The original TikTok video IDs, creator handles, and profile/video URLs are omitted. `record_id` identifies a row within this release, and `creator_group_id` preserves creator grouping for evaluation; the creator ID is not a model feature.

## Columns and feature sets

The CSVs share the same 43-column schema. The model inputs can be reconstructed as follows:

- **Set A:** nine structured features: duration, five posting-time features, and three sound metadata features (`music_originality`, `music_nused`, and `log_music_nused`). Sound-use counts are measured at crawl time, not at posting time.
- **Set B:** Set A plus caption and hashtag text from `vid_desc_clean` and `vid_hashtags_normalized`. Fit TF-IDF and SVD on each training fold only; the SVD columns are intentionally not stored as globally precomputed values.
- **Set C:** Set B plus 14 hook features extracted from the first five seconds: six presence indicators, four segment/movement counts, two quantity measures, and two derived measures.

The `hook_label` column contains seven exploratory labels: `Informational`, `Curiosity`, `Visual Salience`, `Emotional`, `Urgency`, `CTA`, and `Social Proof`. The 989 records that were initially left without a clear label were relabeled with a mandatory best-fit choice among these seven using GPT-4.1-mini. In 12 cases with no usable cue, `Informational` was the required fallback with confidence 1 and blank evidence; this fallback does not establish that a clear informational hook was observed. The LLM-assisted labels are descriptive only, were not validated by human annotators, and are not model inputs. Use `keep_default_na=False` to preserve blank evidence.

## Engagement Score

`engagement_rate_per_100_views_seed42` and `engagement_score_seed42` are supplied for the canonical seed-42 split. The rate is based on likes, comments, shares, and saves per 100 views. Missing shares are imputed using the training-partition median (17 shares for this split), and the rate is clipped at the training-partition 99th percentile (`24.3980153811`) before min-max scaling to 0–100. Raw interaction counts are retained in the files, and missing `share_count` values remain blank with `has_missing_share=1`.

For any other split or seed, recompute the target using only that split’s training partition. Do not use the seed-42 score column as a predictor.

## Reproducing the multi-seed evaluation

To reproduce the five-seed results, concatenate both CSVs, then create a fresh 80/20 creator-disjoint split for each seed: `42`, `123`, `456`, `789`, and `1010`. Recompute target thresholds from each training partition and refit TF-IDF/SVD within every training fold. Use `creator_group_id` as the grouping variable.

## Provenance and license

This dataset accompanies the research manuscript titled above; the manuscript is distributed separately. The source data are based on ViTIED and public TikTok video snapshots. No separate license is specified in this repository.

