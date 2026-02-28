# GAN-Augmented Dual-Stream Late Fusion U-Net for Medical Image Segmentation

##  Overview
This repository contains the official implementation of a hybrid framework that integrates a Generative Adversarial Network (GAN) with a Dual-Stream Late Fusion Segmentation Network. The system is designed to address the challenges of soft-tissue segmentation in Computed Tomography (CT) scans, specifically targeting the brainstem. By synthetically generating the missing MRI modality, the model significantly enhances segmentation performance and accuracy.

##  Architecture

### Proposed Dual-Stream Late Fusion Network
![Proposed Dual-Stream Late Fusion Architecture](image/CT-Trang1.drawio.png)

The proposed workflow consists of three interconnected pipelines:
1. **GAN-based MRI Synthesis:** A Generative Adversarial Network trained on an external unpaired MRI dataset learns the domain adaptation mapping from CT to MRI. During inference, the U-Net based Generator takes preprocessed CT images as input to synthesize the corresponding "Synthetic MRI".
2. **Dual-Stream Late Fusion U-Net:** A segmentation network featuring two specialized parallel encoding branches:
   * **CT Stream (Geometric Encoder):** Processes the original CT data to extract high-fidelity geometric features, such as bony boundaries and rigid landmarks.
   * **sMRI Stream (Semantic Encoder):** Processes the Synthetic MRI to extract soft-tissue semantic features and inter-organ contrast information.
3. **Late Fusion & Decoding:** The distinct feature maps from both streams are merged via channel concatenation at the network's Bottleneck (maintaining a 128x128 spatial dimension) before being upsampled by the Decoder to reconstruct the final predicted mask.

### Baseline Single-Channel U-Net (For Comparison)
![Baseline U-Net Architecture](image/baseline.png)

For performance comparison, a standard single-channel U-Net was implemented to process the CT images directly.

##  Dataset Preparation
The system utilizes two distinct datasets to train the Generative and Segmentation networks separately:
* **Source Domain (Segmentation):** The PDDCA 2015 dataset, comprising 3D CT scans from 48 patients. 
* **Target Domain (GAN Training):** Brain MRI Images dataset, comprising approximately 3,000 unpaired MRI slices to learn the MRI intensity distribution.

##  Loss Function
To overcome extreme class imbalance, the network is optimized using a Hybrid Loss strategy:

$L_{Total} = \lambda_1 L_{BCE} + \lambda_2 L_{Dice}$

This objective function combines Binary Cross Entropy (BCE) to establish a smooth error surface for rapid convergence, and Dice Loss to heavily penalize False Negatives on small target structures, forcing the network to learn detailed boundary features.

##  Experimental Results
The Late Fusion Multi-modal model was evaluated against a single-channel Baseline U-Net over 200 training epochs:

| Metric | Baseline U-Net (CT Only) | Late Fusion U-Net (Proposed) |
| :--- | :--- | :--- |
| **Accuracy** | 99.81% | 99.97% |
| **F1-Score** | 84.38% | 97.50% |
| **Best Dice Score** | 96.58% | 97.50% |
| **Train Loss** | 36.69% | 2.87% |
| **Val Loss** | 39.04% | 3.40% |

**Key Findings:**
* **Faster Convergence:** The proposed model demonstrated exceptional stability, with the training loss dropping rapidly within the first 5 epochs.
* **Enhanced Feature Extraction:** The Fusion model achieved a training loss reduction of approximately 45% compared to the baseline.

### Visual Evaluation
![Segmentation Visual Results](image/case_12_slice_100.png)

Visual evaluations confirm that the multi-modal approach resolves segmentation ambiguities well, maintaining highly accurate predictions for positive cases while effectively demonstrating high specificity on negative cases.
