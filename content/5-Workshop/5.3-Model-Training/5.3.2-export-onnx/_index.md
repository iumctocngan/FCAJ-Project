---
title: "Export Static ONNX Model"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 5.3.2 </b> "
---

#### Export ONNX Model

#### 1. Why Export to ONNX Format?
- Size optimization for Serverless: The uncompressed PyTorch runtime (torch + torchvision) exceeds 1.2 GB, far beyond practical AWS Lambda container sizes.
- Compact footprint and high speed: The ONNX Runtime library requires only 18 MB. When packaged with the food_model.onnx file (15.52 MB), total Lambda package size is approximately ~34 MB, enabling rapid inference execution and optimized cold start latency.

#### 2. Execute ONNX Export Code (Cell 8):

![onnx-export-success](/FCAJ-Project/images/5-Workshop/5.3-S3-vpc/onnx_export_success.png)

#### 3. Acceptance Verification & Local Download:
- After export, verify that food_model.onnx reaches the target size of 15.52 MB.
- Download food_model.onnx (15.52 MB) and calorie_map.json locally for Docker packaging and upload to Amazon S3.
