# Hunt Down Pneumothorax
### Pneumothorax Detection in Wireless Handheld Ultrasound with Artificial Intelligence
**緝胸 — 人工智慧之無線手持超音波氣胸診斷**

**English** | [日本語](README.ja.md)

![Hunt Down Pneumothorax poster](<【副本】緝胸-人工智慧之無線手持超音波氣胸診斷.png>)

**Advisor:** Prof. 吳明龍
**Team:** 錢曉琦, 陳映伃 (Ying-Yu Chen), 胡莉苓, 劉容瑜

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

### Takeaway
The results suggest that this technique is a feasible way to identify pneumothorax in emergency and urgent care, with the potential to **reduce diagnosis time and lower the medical costs of misdiagnosis**.
