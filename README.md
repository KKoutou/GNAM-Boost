# GNAM-Boost Simulation — COVID-19 In-Hospital Mortality (SIVEP-Gripe)  
GNAM-Boost シミュレーション — ブラジル SIVEP-Gripe 入院死亡リスク予測

## 概要 / Overview
本リポジトリは、論文の Simulation 章をそのまま再現できるノートブック一式です。データの読み込み・前処理、分層サンプリング、ベースライン（Logistic / XGBoost）、GNAM、GNAM-Boost の学習と確率校正、評価、図表作成までを含みます。  
This repository reproduces the Simulation section of our study: data import/cleaning, stratified splitting, baselines (Logistic/XGBoost), GNAM, GNAM-Boost training with probability calibration, evaluation, and figure generation.

---

## データ / Data
- 取得先 / Download: https://opendatasus.saude.gov.br/dataset/srag-2020  
- データセット / Dataset: SIVEP-Gripe（SRAG 2020、COVID-19 サブセット）。CSV は「;」区切り。  
- 利用許諾 / License: **改変不可（不可修改 / No derivatives）**。派生物の作成や内容改変は許可されません。最新の条件は配布元（OpenDataSUS）の規約をご確認ください。  
- 本リポジトリは生データを同梱しません。`Import&processing.ipynb` の先頭セルで以下を設定してください：  
  - `CFG.CSV_PATH`：ローカルの元データ CSV パス  
  - `CFG.OUT_DIR`：出力先（既定: `gnamboost_outputs`）

---

## 環境 / Environment
Python ≥ 3.10 を推奨。主要依存はノートブック内のインストール行と一致します。  
Recommended Python ≥ 3.10. Install dependencies:

```bash
pip install numpy pandas scikit-learn xgboost==2.0.3 torch matplotlib pyyaml scipy joblib pillow shap
```

> `OneHotEncoder(min_frequency=...)` が未対応の環境では自動的に通常 OHE にフォールバックします。  
> If your sklearn version lacks `min_frequency`, the code automatically falls back to standard OHE.

---

## ファイル構成 / Repository Layout

```text
.
├─ Import&processing.ipynb     # 取込・クリーニング・文脈特徴量・分層分割・前処理
├─ Model_Training.ipynb        # Logistic / XGBoost / GNAM / GNAM-Boost 学習・校正・CV・表
├─ Plot.ipynb                  # ROC/PR/DCA・GNAM 形状関数・残差SHAP・主/残差の寄与分解
└─ gnamboost_outputs/
   ├─ table1_cohort_summary.csv
   ├─ table1_missingness_full.csv
   ├─ table2_model_compare_metrics.csv
   └─ figs/
      ├─ fig_triptych_roc_pr_dca.png
      ├─ shapes/shape_triptych_overload_epiweek_age.png
      ├─ fig2_residual_shap_overload_epiweek_age_panel.png
      └─ fig_gnamboost_effect_sizes_full_grid.png
```

---

## 実行手順 / How to Run

1. **Import&processing.ipynb**  
   - `UF` の統一、日付の変換、流行週 (`EPI_WEEK`) の生成。  
   - 病床逼迫度 `Overload`（8 週ローリング中位数の 1 週ラグを基準化しクリップ）と、4 週遅延の入院数/致死率代理（`CTX_ADM_4w`、`CTX_CFR_4w`）を病院/州レベルで構築。  
   - ラベルを二値化（生存=1、死亡=2）。  
   - **6 軸分層**（年齢四分位×州×合併症負担 0/1/≥2×教育×人種×性別）。25 件未満の希少層は RARE に統合し、逐層ランダムに分割：  
     - Train 196,576（約 64%）、Valid 49,109（約 16%）、Test 61,390（約 20%）  
     - 事件率：Train 34.68%、Valid 34.27%、Test 34.16%  
     - バランス：最大標準化差（SMD）— Train–Test 0.006、Train–Valid 0.007  
   - 前処理：数値=中央値補完+欠損指示+標準化、二値=最頻値補完、カテゴリ=最頻値補完+One-Hot（未知無視・低頻度統合）。

2. **Model_Training.ipynb**  
   - Logistic（LBFGS）、XGBoost（hist / lossguide、早期打切り）、GNAM（ExU、AdamW、早期打切り）、GNAM-Boost（GNAM ロジットを `base_margin` に設定した残差ブースト）を学習。  
   - 検証集合で等距回帰（isotonic）による確率校正、テスト集合で AUROC / PR-AUC / Brier / 校正傾き・切片 / E:O を評価。  
   - 10-fold CV（XGBoost 基線）で安定性を確認し、表として出力（`table2_model_compare_metrics.csv`）。

3. **Plot.ipynb**  
   - ROC / PR / Decision Curve 三連図、GNAM 形状関数、残差 TreeSHAP、主効果＋残差の寄与スタックを作図・保存。

---

## 再現のための要点 / Key Numbers for Reproducibility
- 分割規模 / Split sizes: Train **196,576**、Valid **49,109**、Test **61,390**  
- 事件率 / Event rates: Train **34.68%**、Valid **34.27%**、Test **34.16%**  
- バランス / Balance: max-SMD — Train–Test **0.006**、Train–Valid **0.007**  
詳細指標は `gnamboost_outputs/table2_model_compare_metrics.csv` を参照。

---

## 注意事項 / Notes
- すべての前処理器は **訓練集合でのみ fit** し、検証/テスト集合には **transform のみ** を適用（情報リーク防止）。  
- カテゴリ高基数や低頻度が多い場合は `min_frequency` の引き上げや列の見直しを検討してください。  
- Windows では `CFG.CSV_PATH` を実ファイルに合わせて絶対パスで設定してください。  
- GNAM は CPU 既定。GPU を使用する場合は PyTorch のデバイス設定を切り替えてください。

---

## 引用 / Citation
- データ出典 / Data source: OpenDataSUS — SIVEP-Gripe（SR
