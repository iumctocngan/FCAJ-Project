---
title: "Week 9 Worklog"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---


### Week 9 Objectives:
- Build the serverless inference backend on AWS Lambda using Docker Container Images.
- Integrate S3 storage, Rekognition AI fallback, configure API Gateway, and deploy the Web UI to AWS Amplify.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date |
| --- | --- | --- | --- |
| 2 | - Build Python Lambda inference using ONNX Runtime with image preprocessing and portion scaling | 07/09/2026 | 07/09/2026 |
| 3 | - Package backend and ONNX model into a Docker container image, push to Amazon ECR, and configure Lambda | 08/09/2026 | 08/09/2026 |
| 4 | - Set up S3 bucket for image storage; integrate Amazon Rekognition as AI fallback when confidence < 60% | 09/09/2026 | 09/09/2026 |
| 5 | - Configure Amazon API Gateway REST API (POST /predict, CORS, Rate Limiting) integrated with Lambda | 10/09/2026 | 10/09/2026 |
| 6 | - Develop responsive Web UI, connect API Gateway, and deploy frontend to AWS Amplify Hosting | 11/09/2026 | 11/09/2026 |

### Week 9 Achievements:
- Successfully deployed ONNX inference backend on AWS Lambda using container images from Amazon ECR.
- Completed POST /predict API with input validation, CORS, rate limiting, and portion-based nutrition responses.
- Integrated S3 and Amazon Rekognition as automated AI fallback for out-of-scope or low-confidence food images.
- Hosted Web UI on AWS Amplify and verified the end-to-end flow from image upload to nutrition display.
