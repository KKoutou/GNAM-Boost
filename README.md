# GNAM–Boost: 先天性異常リスクのための解釈可能機械学習フレームワーク

ブラジル SINASC 出生データを用いて Generalized Neural Additive Model（GNAM）と XGBoost 残差ブースティングを組み合わせた **GNAM–Boost** を実装したリポジトリです。  
日本のエコチル調査（JECS）への応用を想定した，先天性異常リスク予測のための概念実証シミュレーションを含みます。

This repository implements **GNAM–Boost**, an interpretable ensemble that combines a Generalized Neural Additive Model (GNAM) with a residual XGBoost component.  
The current simulation uses Brazilian SINASC live-birth data as a proof of concept for congenital anomaly risk modeling and is designed to be transferable to the Japan Environment and Children’s Study (JECS).

---

## 1. 概要 / Overview

**日本語**

- GNAM が各特徴量の滑らかな主効果（非線形リスクカーブ）を学習し，  
  XGBoost が GNAM では説明しきれない残差構造を補正します。
- ターゲットは「先天性異常あり vs なし」の二値アウトカムで，有病率は約 1% の希少事象です。
- 以下の 4 モデルを同じ前処理・分割条件で比較します：
  1. 簡略化した特徴量セットのロジスティック回帰  
  2. GNAM 単独モデル  
  3. XGBoost 単独モデル  
  4. GNAM–Boost（GNAM + XGBoost 残差）

**English**

- GNAM models smooth, feature-wise main effects (non-linear risk curves).  
  XGBoost is trained on residuals to capture additional non-linear and interaction structure.
- The target outcome is a binary label: *any congenital anomaly vs none*, with prevalence ≈ 1%.
- Four models are compared under the same preprocessing and data split:
  1. Logistic regression with a simplified feature set  
  2. GNAM only  
  3. XGBoost only  
  4. GNAM–Boost (GNAM + residual XGBoost)

---

## 2. 特徴 / Features

**日本語**

- 母親年齢・妊娠週数・出生体重などに対する **GNAM 主効果カーブ**を可視化。
- XGBoost 残差モデルにより，交互作用や局所的な非線形パターンを補足。
- ROC, Precision–Recall カーブ，Decision Curve Analysis による性能評価。
- GNAM 主効果（緑）と XGBoost 残差寄与（赤）を積み上げて表示した特徴重要度プロット。

**English**

- GNAM main-effect curves for key predictors (maternal age, gestational age, birth weight).
- Residual XGBoost component to recover interaction and higher-order non-linear patterns.
- Evaluation via ROC, precision–recall curves, and decision-curve analysis.
- Stacked importance plots showing GNAM main effects (green) and residual contributions (red).

---

## 3. ノートブック構成 / Notebooks

リポジトリの主な解析フローは，以下 3 つの Jupyter Notebook で構成されています。

**日本語**

1. `Import&processing.ipynb`  
   - SINASC の `DN2024.dbc` を読み込み，必要な変数の選択・前処理・欠損処理・学習用データフレームの作成を行います。

2. `Model_Training.ipynb`  
   - ロジスティック回帰，GNAM，XGBoost，GNAM–Boost の 4 モデルを学習・評価します。  
   - 学習済みモデルや中間結果を保存します。

3. `Plot.ipynb`  
   - Fig. 2 に相当する図の作成：  
     ROC / PR カーブ，Decision Curve，GNAM 主効果カーブ，XGBoost 残差寄与，特徴重要度などをプロットします。

**English**

1. `Import&processing.ipynb`  
   - Load `DN2024.dbc` from SINASC, select variables, perform preprocessing and missing-data handling, and construct the analysis dataset.

2. `Model_Training.ipynb`  
   - Train and evaluate the four models: logistic regression, GNAM, XGBoost, and GNAM–Boost.  
   - Save fitted models and intermediate outputs.

3. `Plot.ipynb`  
   - Generate figures corresponding to Fig. 2 in the proposal, including ROC/PR curves, decision curves, GNAM main-effect plots, XGBoost residual contributions, and stacked importance plots.

---

## 4. 必要環境 / Requirements


- Python 3.9 以上  
- PyTorch（GNAM 実装用）  
- XGBoost  
- scikit-learn  
- numpy, pandas, matplotlib などの基本的な科学計算ライブラリ

`requirements.txt` がある場合は，以下でインストールできます。

```bash
pip install -r requirements.txt
