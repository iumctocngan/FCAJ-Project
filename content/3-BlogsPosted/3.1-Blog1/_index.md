---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# MLOps on AWS SageMaker: From Model Training to Continuous Monitoring

> Training a model with high accuracy is only the first step. When taking a model to production, we must track which data it was trained on, which code version produced it, which model version is currently deployed, and whether model performance degrades over time. This is precisely the problem that **MLOps** solves.

On **AWS**, an end-to-end MLOps pipeline can be built around **SageMaker Pipelines**, **Model Registry**, **Baseline/Drift Checking**, and **Model Monitoring** tools.

---

### 1. What Does MLOps Need to Control?

Unlike traditional software engineering where behavior is primarily defined by source code, **Machine Learning (ML)** systems continuously evolve across three dimensions: **code**, **data**, and **real-world environment behavior**.

There are three primary areas to govern:

1. **Code & Configuration Changes:** Modifications in data preprocessing, hyperparameters, dependencies, or model architecture. Using Git, artifact versioning, and lineage tracking helps pinpoint exactly which code generated each model version.
2. **Data Drift:** The statistical distribution of production input data shifts compared to the baseline training data.
3. **Concept Drift:** The underlying relationship between input features and target outputs changes over time, leading to performance degradation even when input distributions look stable.

> [!IMPORTANT]
> **Key Insight:** The occurrence of *drift* does not automatically mean the model is broken. Rather, drift serves as an **early warning signal** to inspect data quality and model performance before making informed retraining decisions.

---

### 2. SageMaker Pipelines and Baseline Management

**Amazon SageMaker Pipelines** is a purpose-built CI/CD service for ML workflows, orchestrating each step: data preprocessing, model training, evaluation, and registration.

A crucial component in the pipeline is the **Baseline**. A baseline consists of reference statistics and constraints calculated from baseline training/validation datasets, against which future production data is compared.

SageMaker provides dedicated pipeline steps:
1. **`QualityCheckStep`**: Calculates baseline statistics and validates data quality and model prediction quality.
2. **`ClarifyCheckStep`**: Computes baselines for data bias, model fairness, and explainability metrics.

Two essential configuration flags:
1. **`skip_check`**: Determines whether to skip comparison against the existing reference baseline.
2. **`register_new_baseline`**: Determines whether the newly calculated baseline should be registered as the new reference for subsequent pipeline runs.

*(On the initial run without prior baselines, the pipeline generates the baseline. Subsequent runs compare newly ingested data against this established baseline).*

---

### 3. SageMaker Model Registry and Governance

Upon completing training and evaluation, models should not be deployed directly to production without proper governance and approval.

**SageMaker Model Registry** centrally manages model versions, metadata, metrics, and lineage tracking. Each model version is assigned an approval status:

1. `PendingManualApproval`: Default status after training, awaiting review from ML engineers or QA.
2. `Approved`: Model satisfies technical and business standards, triggering automated deployment workflows.
3. `Rejected`: Model fails criteria and is barred from deployment.

> [!TIP]
> **Lineage & Traceability:** Baseline statistics and evaluation reports can be directly associated with specific model versions in the Model Registry, providing complete auditability from raw data to deployed endpoints.

---

### 4. Continuous Monitoring and Drift Detection

After deployment to real-time endpoints or serverless inference, continuous monitoring is maintained via **Amazon SageMaker Model Monitor**:

1. **Capture Real-Time Payload Logs:** Ingest input features and model predictions to Amazon S3.
2. **Compare Against Baselines:** Periodically run monitoring schedules to detect statistical data drift.
3. **Ground-Truth Evaluation:** When delayed ground-truth labels become available, recalculate performance metrics (*Accuracy, Precision, Recall, F1-Score*) to detect model quality degradation.

**Standard drift handling workflow:**
```
Detect Drift / Alarm Triggered
       │
       ▼
Investigate Root Cause (Seasonality, data pipeline bug, user behavior change)
       │
       ▼
Collect & Validate Fresh Datasets
       │
       ▼
Trigger Retraining Pipeline
       │
       ▼
Evaluate & Approve in Model Registry
       │
       ▼
Deploy New Version (Safe Canary / Blue-Green Rollout)
```

> [!CAUTION]
> **Avoid Blind Automated Retraining:** Do not automatically trigger retraining and immediate deployment upon detecting drift. Drift can stem from transient events (e.g., promotional campaigns) or upstream data corruption. Human or rule-based root cause analysis should precede retraining.

---

### 5. Is Amazon SageMaker Feature Store Mandatory?

**SageMaker Feature Store** provides a centralized repository for curated ML features, offering dual storage layers:
- **Online Store:** Ultra-low latency retrieval (single-digit ms) for real-time inference.
- **Offline Store:** Historical feature storage on Amazon S3 for batch training and point-in-time time-travel queries.

#### When to Use:
- Highly valuable for tabular workloads with complex feature transformations, such as *Fraud Detection*, *Recommendation Engines*, and *Demand Forecasting*.

#### Practical Considerations:
- Feature Store **does not automatically eliminate** *Training-Serving Skew*. Engineers must ensure consistent transformation logic across training and serving paths.
- **Not required for every workload:** For computer vision (CV) or natural language processing (NLP) applications where raw files/images reside directly in S3, adding Feature Store can introduce unnecessary complexity and cost.

---

### 6. End-to-End MLOps Lifecycle

The complete SageMaker MLOps lifecycle can be summarized as a closed-loop flow:

```
Data → Processing → Training → Evaluation → Quality/Drift Check
  ↓
Model Registry
  ↓
Approval
  ↓
Deployment
  ↓
Monitoring
  ↓
Collect New Data
  ↓
Retraining
```

In summary, **MLOps is much more than automated Train → Deploy scripts**. Its true purpose is to govern the **entire lifecycle of the model**: data, code, versioning, evaluation, approval, deployment, monitoring, and continuous feedback.

---

### 7. Conclusion

A robust MLOps system must definitively answer **three key questions**:
1. *Where did the production model come from (data lineage, code commit)?*
2. *Why was this specific model approved for deployment (metrics, approvals)?*
3. *Is the model continuing to perform reliably in production?*

AWS SageMaker provides a comprehensive suite of native services to fulfill these requirements. Start with a lean, manageable pipeline suited to your current workload, and progressively introduce Model Registry, Drift Detection, and Feature Store as your system scales.

---

### References

1. [Amazon SageMaker AI — Pipelines Overview](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines-overview.html)
2. [Amazon SageMaker AI — Baseline & Drift Detection](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines-quality-clarify-baseline-lifecycle.html)
3. [Amazon SageMaker AI — Model Registry](https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry.html)
4. [Amazon SageMaker AI — Feature Store Concepts](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store-concepts.html)
