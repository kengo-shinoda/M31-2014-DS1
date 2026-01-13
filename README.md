# Failed supernova identification and interpretation of M31-2014-DS1 and NGC 6946-BH1

Code and data to reproduce results on the disappearance of M31-2014-DS1 and its interpretation with NGC 6946-BH1 as a failed supernova. 

## Data
All the photometry of the object is provided in photometry.csv. The photometry include flux measurements and their errors in epochs where the source was detected; otherwise the flux error column is empty and the is_limit flag is set to 1. 

The raw MMT/Binospec data can be found in the directory raw_data/

## Analysis
The py_zogy.py script (taken from Frank Masci's implementation at https://github.com/cenko/ZOGY/blob/master/masci/py_zogy.py) performs image subtraction and photometry on the NEOWISE images.

## Models
The MESA models for both M31-2014-DS1 and NGC 6946-BH1 are in the folder mesaruns/ 
The python notebook MESAfailedSN.ipynb uses the MESA model outputs to reproduce figures for the interpretation.

## M31-2014-DS1 の H envelope が剥がれる仕組み（コード構造ベース）
以下は `mesaruns/m31_hpoor/` を読み解いて、H envelope が星の進化で薄くなる仕組みをまとめたものです。

- 実行フローは段階的な MESA ランで、保存モデルを引き継ぐ構成です。`inlist_to_zams` -> `inlist_to_end_core_h_burn` -> `inlist_to_end_core_he_burn` -> `inlist_to_end_core_c_burn` の順に `load_saved_model` / `save_model_filename` を使って進化を継続します。
- 風による質量損失は `inlist_common` の wind 設定（Dutch 系スキーム）を基に、`inlist_to_end_core_h_burn` と `inlist_to_end_core_he_burn` で `Dutch_scaling_factor = 1.0` として通常の風を有効化しています。
- H-poor 化の主因は後期の強制的な質量損失です。`inlist_to_end_core_c_burn` で `hot_wind_scheme` / `cool_wind_*` を空にして風スキームを無効化し、その代わり `mass_change = -2.376e-4` を指定して一定の質量減少率を課しています。`history.data` の `log_abs_mdot = -3.624` はこの値（約 2.38e-4 Msun/yr）と対応します。
- `src/run_star_extras.f90` では `deltaHmass_other_timestep_limit` が定義され、`star_mass_h1`（H envelope 質量）と `star_mdot` を使って timestep を制限します。`x_ctrl(1)`（inlist で指定）により、H envelope の減少が粗くならないように時間刻みが自動調整されます。
- 具体的な結果は `mesaruns/m31_hpoor/LOGS/history.data` にあり、`total_mass_h1` が 4.29 Msun から 0.157 Msun に減少し、`star_mass` も 10.87 Msun から 5.02 Msun まで落ちています。これが H-poor envelope の実体です。

論文との対応:
- 本文 Figure 2 では M31-2014-DS1 の SED から薄い H envelope（約 0.55 Msun）が示されており、上記の H-poor モデルと整合します。
- Supplementary Figure S5 は H-poor / H-rich の envelope 構造比較で、H-poor 側がより中心集中であることを示しています。
- Supplementary Eq. S7: `v_sh = v0 * (r / R_*)^alpha`（shock 速度の簡略モデル）。直接の MESA 進化ではなく、崩壊後の envelope 応答モデルで使われる式です。
