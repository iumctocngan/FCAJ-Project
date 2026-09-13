---
title: "Model Training on Colab"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

#### AI Training Steps on Google Colab GPU (Stage 1)

1. Open notebook and check environment (Cell 1):
   - Open notebook file [Food101_FineTuning_Colab.ipynb](https://colab.research.google.com/drive/1sLn-Jui3FGhMCUTgPHQDb6mu1PsESRzX?usp=sharing) on Google Colab.
   - Execute Cell 1 to verify T4 GPU settings and install dependencies.

2. Download dataset and create calorie_map (Cell 2 & 3):
   - Execute Cell 2 to download Food-101 and extract the 50 most popular food classes (50,000 images: Train 37,500 | Val 7,500 | Test 5,000).
   - Execute Cell 3 to generate the calorie_map.json lookup file.

![colab-training](/FCAJ-Project/images/5-Workshop/5.3-S3-vpc/dataset.png)

3. Configure and train EfficientNet-B0 (Cell 4, 5 & 6):
   - Configure 288x288 input with TrivialAugmentWide data augmentation.
   - Stage 1 (Warmup): Freeze backbone for 3 epochs (Lr = 1e-3).
   - Stage 2 (Full Unfreeze): Train full backbone 7 epochs with Discriminative LR (Backbone 1e-4, Classifier 1e-3) and Cosine Annealing.

![colab-training](/FCAJ-Project/images/5-Workshop/5.3-S3-vpc/colab_training.png)

4. Verify results (Cell 7):
   - Test evaluation on 5,000 test set images achieves 85.62% Test Top-1 Accuracy and 0.86 F1-score.
   - Checkpoint saved automatically as best_food_model.pth (15.8 MB).

![colab-training](/FCAJ-Project/images/5-Workshop/5.3-S3-vpc/accuracy.png)
![colab-training](/FCAJ-Project/images/5-Workshop/5.3-S3-vpc/f1.png)