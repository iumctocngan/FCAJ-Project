---
title: "Week 10 Worklog"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---


### Week 10 Objectives:

- Prepare the dataset and build NutriVision's food recognition model.
- Evaluate and optimize the model, then prepare artifacts for backend integration.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date |
| --- | --- | --- | --- |
| 2 | - Explore Food-101 and select 50 commonly consumed food classes | 14/09/2026 | 14/09/2026 |
| 3 | - Clean the data, create train/validation/test splits, and build the image preprocessing pipeline | 15/09/2026 | 15/09/2026 |
| 4 | - Fine-tune EfficientNet-B0 on Google Colab GPU and monitor training | 16/09/2026 | 16/09/2026 |
| 5 | - Evaluate Accuracy, Precision, Recall, and F1-score; analyze the confusion matrix and prediction errors | 17/09/2026 | 17/09/2026 |
| 6 | - Export the model to ONNX, run local inference tests, and finalize calorie_map.json | 18/09/2026 | 18/09/2026 |

### Week 10 Achievements:

- Completed the 50-class dataset and a consistent preprocessing pipeline for training and testing.
- Successfully fine-tuned EfficientNet-B0 and evaluated it on an independent test set.
- Exported a 15.5 MB ONNX model with 85.62% Test Top-1 Accuracy and a 0.86 Weighted F1-Score.
- Finalized calorie_map.json and verified local inference results for backend integration.
