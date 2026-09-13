---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# MLOps on AWS SageMaker: From Model Training to Continuous Monitoring

> Training a model with high accuracy is only the first step. When taking a model to production, we must track which data it was trained on, which code version produced it, which model version is currently deployed, and whether model performance degrades over time. This is precisely the problem that MLOps solves.

On AWS, an end-to-end MLOps pipeline can be built around SageMaker Pipelines, Model Registry, baseline/drift checking, and Model Monitor tools.

---

### 1. What Does MLOps Need to Control?

Unlike traditional software engineering where behavior is primarily defined by source code, Machine Learning systems continuously evolve across three dimensions: code, data, and real-world environment behavior.

Three primary areas must be managed:

- Code and Configuration Changes: Modifications in data preprocessing, hyperparameters, dependencies, or model architecture. Using Git, artifact versioning, and lineage tracking helps pinpoint exactly which code generated each model version.
- Data Drift: The statistical distribution of production input data shifts compared to the baseline training data.
- Concept Drift: The underlying relationship between input features and target outputs changes over time, leading to performance degradation even when input distributions appear normal.

> [!IMPORTANT]
> The occurrence of drift does not automatically mean the model is broken. Rather, drift serves as an early warning signal to inspect data quality and model performance before making informed retraining decisions.

---

### 2. SageMaker Pipelines and Baseline Management

Amazon SageMaker Pipelines is a purpose-built CI/CD service for ML workflows, orchestrating each step: data preprocessing, model training, evaluation, and registration.

A crucial component in the pipeline is the baseline. A baseline consists of reference statistics and constraints calculated from baseline training/validation datasets, against which future production data is compared.

SageMaker provides dedicated pipeline steps:
- QualityCheckStep: Calculates baseline statistics and validates data quality and model prediction quality.
- ClarifyCheckStep: Computes baselines for data bias, model fairness, and explainability metrics.

Two common configuration parameters:
- skip_check: Determines whether to skip comparison against the existing reference baseline.
- register_new_baseline: Determines whether the newly calculated baseline should be registered as the new reference for subsequent pipeline runs.

*(On the initial run without prior baselines, the pipeline generates the baseline. Subsequent runs compare newly ingested data against this established baseline).*

---

### 3. SageMaker Model Registry and Governance

Upon completing training and evaluation, models should undergo review and governance before being used in production.

SageMaker Model Registry centrally manages model versions, metadata, metrics, and lineage tracking. Each model version is assigned an approval status:

- PendingManualApproval: Default status after training, awaiting review from engineers or QA.
- Approved: Model satisfies technical and business criteria, triggering automated deployment workflows.
- Rejected: Model fails criteria and is barred from deployment.

> [!TIP]
> Lineage and Traceability: Baseline statistics and evaluation reports can be directly associated with specific model versions in the Model Registry, providing complete auditability from raw data to deployed endpoints.

---

### 4. Continuous Monitoring and Drift Detection

After deployment to real-time endpoints or serverless inference, continuous monitoring is maintained via Amazon SageMaker Model Monitor:

- Capture Real-Time Logs: Ingest input features and model predictions to Amazon S3.
- Compare Against Baselines: Periodically run monitoring schedules to detect statistical data drift.
- Ground-Truth Evaluation: When ground-truth labels become available later, recalculate performance metrics (Accuracy, Precision, Recall, F1-Score) to detect model quality degradation.

Standard drift handling flow:
Detect Drift or Alarm Triggered ➔ Investigate Root Cause (seasonality, pipeline bug, user behavior change) ➔ Collect and Validate Fresh Datasets ➔ Trigger Retraining Pipeline ➔ Evaluate and Approve in Model Registry ➔ Deploy New Version.

> [!CAUTION]
> Avoid Blind Automated Retraining: Do not automatically trigger retraining and immediate deployment upon detecting drift. Drift can stem from transient events or upstream data corruption. Root cause analysis should precede retraining.

---

### 5. Is Amazon SageMaker Feature Store Mandatory?

SageMaker Feature Store provides a centralized repository for curated ML features, offering dual storage layers:
- Online Store: Low latency retrieval (single-digit ms) for real-time inference.
- Offline Store: Historical feature storage on Amazon S3 for batch training and point-in-time queries.

#### When to Use:
Valuable for tabular workloads with complex feature transformations, such as fraud detection, recommendation engines, and demand forecasting.

#### Practical Considerations:
Feature Store does not automatically eliminate Training-Serving Skew; engineers must ensure consistent transformation logic across training and serving paths. Additionally, it is not required for every workload: for computer vision or NLP applications where raw files/images reside directly in S3, adding Feature Store can introduce unnecessary complexity and cost.

---

### 6. End-to-End MLOps Lifecycle

The complete SageMaker MLOps lifecycle can be summarized as:

Data ➔ Processing ➔ Training ➔ Evaluation & Baseline/Drift Check ➔ Model Registry ➔ Approval ➔ Deployment ➔ Monitoring ➔ Collect New Data ➔ Retraining.

In summary, MLOps is much more than automated training and deployment scripts. Its purpose is to govern the entire lifecycle of the model: data, code, versioning, evaluation, approval, deployment, monitoring, and continuous feedback.

---

### 7. Conclusion

A robust MLOps system should answer three key questions:
1. Where did the production model come from (data lineage, code commit)?
2. Why was this specific model approved for deployment (metrics, approvals)?
3. Is the model continuing to perform reliably in production?

AWS SageMaker provides a comprehensive suite of native services to fulfill these requirements. Teams can start with a lean pipeline suited to their current needs, then progressively adopt Model Registry, Drift Detection, and Feature Store as systems scale.

---

### References

1. [Amazon SageMaker AI — Pipelines Overview](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines-overview.html)
2. [Amazon SageMaker AI — Baseline & Drift Detection](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines-quality-clarify-baseline-lifecycle.html)
3. [Amazon SageMaker AI — Model Registry](https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry.html)
4. [Amazon SageMaker AI — Feature Store Concepts](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store-concepts.html)
