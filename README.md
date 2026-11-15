
# Hunt Down Pneumothorax
### Pneumothorax Detection in Wireless Handheld Ultrasound with Artificial Intelligence
**緝胸 — 人工智慧之無線手持超音波氣胸診斷**
Please see the **English and Japanese translation** down below.

![Hunt Down Pneumothorax poster](<【副本】緝胸-人工智慧之無線手持超音波氣胸診斷.png>)

**Advisor:** Prof. 吳明龍
**Team:** 錢曉琦, 陳映伃 (Ying-Yu Chen), 胡莉苓, 劉容瑜

**指導教員：** 吳明龍 教授
**メンバー：** 錢曉琦、陳映伃（チェン・イーンユイ）、胡莉苓、劉容瑜

---

## Introduction

- Compared with X-ray and CT, ultrasound involves no ionizing radiation and is portable.
- Wireless handheld ultrasound is well suited to emergency departments and ambulances (eFAST protocol).
- This project combines wireless handheld ultrasound with AI to give a preliminary pneumothorax diagnosis at the point of care.
- Operators only need to record routine **B-mode** (brightness mode) videos. The computer assists with diagnosis, so no extra **M-mode** (motion mode) capture is needed. This makes the device easier to use and saves time.
- The system is intended to make pneumothorax diagnosis faster in blunt-trauma emergency care.

## Materials and Methods

### Pipeline

```
1. Pre-processing  →  2. Feature Extraction  →  3. Modeling  →  4. Computer-Aided Diagnosis
```

#### 1. Pre-processing
- Intensity normalization
- Detect the bat sign and pleural line depth in B-mode images (Fig. 2)
- Generate pseudo M-mode images (Fig. 3)
- Crop the M-mode images to the depth range suitable for pneumothorax detection

#### 2. Feature Extraction
- GLCM (Gray-Level Co-occurrence Matrix), 5 feature types
- A total of 20 features extracted from each pseudo M-mode image

#### 3. Modeling
- 17 subjects (11 normal, 6 pneumothorax), 303 pseudo M-mode images in total
- Three-fold cross-validation (training/testing split of 11/6 or 12/5 subjects)
- LightGBM (Light Gradient Boosting Machine)
  - max depth: 20, num leaves: 100, learning rate: 0.05

#### 4. Computer-Aided Diagnosis
- Five pseudo M-mode images, taken at the center of the pleural line, per testing subject
- Majority voting over these five images gives the final diagnosis

### Fig. 2: Bat Sign & Pleural Line Detection

1. **Step 1:** Compute the sum of pixel intensities for each row (y) of the image.
2. **Step 2:** Find the y-coordinate with the highest cumulative intensity near the middle of the image. This row is the pleural line.
3. **Step 3:** Starting from the Step 2 result, search left and right for the first peak below the average pixel intensity sum. These peaks locate the bat sign.

### Fig. 3: Pseudo M-mode Generation

- From a B-mode video of *n* frames, take the same vertical line (at the pleural line center) from each frame.
- Each frame contributes one column. Placing the *n* columns side by side over time gives a pseudo M-mode image with **time** on the x-axis and **depth** on the y-axis.
- Each image is cropped to **250 × 65 pixels**.

## Analysis Results

The dataset was split so that clinical and simulated data were evenly distributed across the training and testing sets. Three train/test splits were used to train and evaluate the model on distinguishing pneumothorax. As in the final diagnosis, only the five central images were used per subject.

Values are shown **per image (per subject)**.

| Fold | Accuracy | False Positive | False Negative | Support (normal : pneumothorax) |
|:----:|:--------:|:--------------:|:--------------:|:-------------------------------:|
| 1 | 0.90 (1.00) | 3 (0) | 0 (0) | 20 : 10 (4 : 2) |
| 2 | 1.00 (1.00) | 0 (0) | 0 (0) | 20 : 10 (4 : 2) |
| 3 | 1.00 (1.00) | 0 (0) | 0 (0) | 15 : 10 (3 : 2) |

At the subject level, all three folds reached **100% accuracy** with no false positives or false negatives.

### Feature Importance

We compared the training results above with GLCM feature importance to check how the images relate to the features. The important features shared across folds, written as (distance, angle), are:

- (1, 0) Correlation
- (1, 0) Homogeneity
- (3, 0) Homogeneity
- (1, 45) Correlation

## Conclusion

### What this project solves
- Diagnosis works directly from standard B-mode ultrasound videos, with no M-mode scan needed, which makes the system easier to use.
- Combining wireless handheld ultrasound with AI gives computer-aided diagnosis at emergency scenes, helping first responders and saving critical time for patients.
- The model is lightweight and suitable for AI edge computing. Analysis takes about **2 seconds per patient** on average, fast enough for emergency settings.
- Cross-validation gives early evidence of the model's ability to generalize and to support diagnosis.

### Limitations
- Partial pneumothorax is not yet considered.
- Probe movement during handheld scanning makes the videos harder to interpret.
- No large dataset is available yet, so more clinical data is needed for testing.


---

## 研究背景と目的

- 超音波検査は X 線や CT と比べて電離放射線の被ばくがなく、持ち運びも容易です。
- ワイヤレス携帯型超音波は、救急外来や救急車内での使用に適しています（eFAST プロトコル）。
- 本プロジェクトでは、ワイヤレス携帯型超音波と AI を組み合わせ、救急現場で気胸の一次診断を行います。
- 操作者は通常の **B モード**（brightness mode）動画を撮影するだけでよく、コンピュータが診断を補助します。追加で **M モード**（motion mode）画像を撮影する必要がないため、操作が簡単になり、時間も節約できます。
- 鈍的外傷の救急医療における気胸診断の効率向上を目指しています。

## 研究材料と方法

### 処理の流れ

```
1. 前処理  →  2. 特徴抽出  →  3. モデル構築  →  4. コンピュータ支援診断
```

#### 1. 前処理（Pre-processing）
- 輝度の正規化
- B モード画像から bat sign と胸膜ラインの深さを検出（図 2）
- 擬似 M モード画像を生成（図 3）
- 気胸の検出に適した深さの範囲で M モード画像を切り出し

#### 2. 特徴抽出（Feature Extraction）
- GLCM（Gray-Level Co-occurrence Matrix、濃度共起行列）の特徴量 5 種類
- 擬似 M モード画像 1 枚につき計 20 個の特徴量を抽出

#### 3. モデル構築（Modeling）
- 被験者 17 名（正常 11 名、気胸 6 名）、擬似 M モード画像は計 303 枚
- 3 分割交差検証（学習/テスト = 11/6 名または 12/5 名）
- LightGBM（Light Gradient Boosting Machine）
  - max depth：20、num leaves：100、learning rate：0.05

#### 4. コンピュータ支援診断（Computer-Aided Diagnosis）
- テスト被験者 1 名につき、胸膜ラインの中心で取得した擬似 M モード画像 5 枚を使用
- 5 枚の多数決で最終的な診断を決定

### 図 2：Bat Sign と胸膜ラインの検出

1. **Step 1：** 画像の各行（y）について、画素の輝度の合計を計算します。
2. **Step 2：** 画像の中央付近で輝度の累積値が最大になる y 座標を求めます。この行が胸膜ラインです。
3. **Step 3：** Step 2 の結果を中心に左右へ探索し、画素輝度の合計の平均を下回る最初のピークをそれぞれ見つけます。これが bat sign の位置になります。

### 図 3：擬似 M モード画像の生成

- *n* フレームの B モード動画から、各フレームの同じ縦ライン（胸膜ラインの中心）を取り出します。
- 1 フレームが 1 列に対応します。*n* 列を時間順に並べると、横軸が **時間**、縦軸が **深さ** の擬似 M モード画像になります。
- 各画像は **250 × 65 ピクセル** に切り出します。

## 分析結果

臨床データと模擬データが学習用とテスト用に均等に分かれるようにデータセットを分割しました。3 組の学習/テストデータでモデルを学習させ、気胸を識別する能力を評価しました。最終診断と同様に、各被験者について中央の 5 枚のみを使用しています。

数値は **画像単位（被験者単位）** で示しています。

| Fold | 正解率 | 偽陽性 | 偽陰性 | サンプル数（正常 : 気胸） |
|:----:|:------:|:------:|:------:|:-------------------------:|
| 1 | 0.90 (1.00) | 3 (0) | 0 (0) | 20 : 10 (4 : 2) |
| 2 | 1.00 (1.00) | 0 (0) | 0 (0) | 20 : 10 (4 : 2) |
| 3 | 1.00 (1.00) | 0 (0) | 0 (0) | 15 : 10 (3 : 2) |

被験者単位では、3 つの Fold すべてで **正解率 100%** となり、偽陽性・偽陰性はありませんでした。

### 特徴量の重要度

上記の学習結果を GLCM の特徴量重要度と照らし合わせ、画像と特徴量の関係を検証しました。各 Fold に共通して重要だった特徴量は次のとおりです（距離, 角度）。

- (1, 0) Correlation
- (1, 0) Homogeneity
- (3, 0) Homogeneity
- (1, 45) Correlation

## 結論と今後の展望

### 本研究で解決したこと
- 通常の B モード超音波動画を入力するだけで診断でき、M モードの撮影が不要なため、使いやすさが向上しました。
- ワイヤレス携帯型超音波と AI を組み合わせることで、救急現場でコンピュータ支援診断が可能になり、救急隊員を支援し、患者の救命時間を確保できます。
- 軽量なモデルのため AI エッジコンピューティングに適しており、患者 1 名あたりの平均解析時間は **約 2 秒** と、救急現場に求められる速さを満たしています。
- 交差検証により、モデルの汎化性能と診断補助の有効性を初期的に確認しました。

### 課題
- 部分的な気胸（partial pneumothorax）はまだ考慮していません。
- 携帯型プローブの揺れなどにより、動画の判読が難しくなる場合があります。
- 大規模なデータセットがまだないため、さらに多くの臨床データでの検証が必要です。

### まとめ
本研究で開発した手法は、救急・救急外来の臨床現場で気胸を判別するうえで一定の実現可能性を示しました。**診断時間の短縮と誤診による医療コストの削減** という目標に沿う結果です。


### Takeaway
The results suggest that this technique is a feasible way to identify pneumothorax in emergency and urgent care, with the potential to **reduce diagnosis time and lower the medical costs of misdiagnosis**.
