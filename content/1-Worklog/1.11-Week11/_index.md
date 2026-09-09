---
title: "Week 11 Worklog"
date: 2024-01-01
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---


### Week 11 Objectives:

- Build the inference backend and deploy the model on AWS Serverless architecture.
- Develop the web interface and connect all components into a working system.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date |
| --- | --- | --- | --- |
| 2 | - Build Python Lambda inference with image processing, quality checks, and portion-based nutrition calculation | 21/09/2026 | 21/09/2026 |
| 3 | - Package the backend and ONNX model as a Docker image, push it to Amazon ECR, and deploy Lambda | 22/09/2026 | 22/09/2026 |
| 4 | - Create S3 storage for artifacts and low-confidence images; integrate Amazon Rekognition as the AI fallback | 23/09/2026 | 23/09/2026 |
| 5 | - Configure API Gateway POST /predict, CORS, rate limiting, and test the Lambda integration | 24/09/2026 | 24/09/2026 |
| 6 | - Complete the Web UI, connect the API, and deploy the frontend to AWS Amplify Hosting | 25/09/2026 | 25/09/2026 |

### Week 11 Achievements:

- Deployed the ONNX inference backend to AWS Lambda using a Docker Container Image from Amazon ECR.
- Completed POST /predict with input validation, CORS, rate limiting, and portion-based nutrition responses.
- Integrated Amazon S3 for artifacts and OOD images, plus Amazon Rekognition as fallback below 60% confidence.
- Hosted the Web UI on AWS Amplify and verified the end-to-end flow from image upload to displayed results.
