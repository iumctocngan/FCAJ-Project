---
title: "Deploy AWS Lambda Container Image"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

#### Containerize AI Inference via Docker & Deploy AWS Lambda using AWS CLI

Food classification inference relies on native C++ libraries (onnxruntime, numpy, and Pillow). To ensure reliable and reproducible execution in the cloud, package your application source code and model artifacts into a **Docker Container Image** and deploy it to **AWS Lambda (PackageType Image)** using the **AWS CLI**.

---

#### 1. Prepare Build Directory & Dockerfile

Ensure **Docker Desktop** is up and running on your local machine.

Verify that directory [backend/deploy/](https://github.com/iumctocngan/NutriVision/tree/main/backend/deploy) contains the necessary files:
- **Dockerfile**
- **requirements.txt**
- [lambda_function.py](https://github.com/iumctocngan/NutriVision/blob/main/backend/deploy/lambda_function.py)
- **food_model.onnx** (15.5 MB)
- **calorie_map.json**

Content of **Dockerfile**:
```dockerfile
FROM public.ecr.aws/lambda/python:3.12

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY lambda_function.py food_model.onnx calorie_map.json ./

CMD [ "lambda_function.lambda_handler" ]
```

> [!TIP] Cold Start Optimization:
> Directly bundling **food_model.onnx** and **calorie_map.json** into the image enables Lambda to load the model locally within ~1 second, eliminating network latency compared to fetching from S3 continuously.

---

#### 2. Create Amazon ECR Repository & Authenticate Docker

```bash
aws ecr create-repository --repository-name nutrivision-lambda --region ap-southeast-1

aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 110359221458.dkr.ecr.ap-southeast-1.amazonaws.com
```

---

#### 3. Build Docker Image & Push to Amazon ECR

Navigate to the `backend/deploy` directory and build:

```bash
cd backend/deploy

docker build --platform linux/amd64 --provenance=false -t nutrivision-lambda:latest .

docker tag nutrivision-lambda:latest 110359221458.dkr.ecr.ap-southeast-1.amazonaws.com/nutrivision-lambda:latest

docker push 110359221458.dkr.ecr.ap-southeast-1.amazonaws.com/nutrivision-lambda:latest
```

![Amazon ECR Image Pushed](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/lambda_ecr_image_pushed.png)

---

#### 4. Configure IAM Permissions & Amazon Rekognition Fallback Integration

Amazon Rekognition is a Serverless Managed AI API operating on-demand, which **requires no server provisioning or complex manual console setup**.

1. **Attach S3 and Amazon Rekognition access policies to Lambda IAM execution role:**
```bash
aws iam attach-role-policy --role-name NutriVisionPredictor-role-kzg0ihpv --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam attach-role-policy --role-name NutriVisionPredictor-role-kzg0ihpv --policy-arn arn:aws:iam::aws:policy/AmazonRekognitionFullAccess
```

2. **Amazon Rekognition SDK Integration Code in [lambda_function.py](https://github.com/iumctocngan/NutriVision/blob/main/backend/deploy/lambda_function.py):**
The Lambda function invokes AWS Boto3 SDK to automatically trigger Rekognition whenever the primary ONNX model confidence drops below 60%:
```python
REKOGNITION_CLIENT = boto3.client("rekognition", region_name="ap-southeast-1")

def detect_labels_fallback(image_bytes, database, classes):
    response = REKOGNITION_CLIENT.detect_labels(
        Image={"Bytes": image_bytes},
        MaxLabels=15,
        MinConfidence=60.0
    )
    labels = [item.get("Name", "") for item in response.get("Labels", [])]
    
    for label in labels:
        matched_key = find_rekognition_match(label, database, classes)
        if matched_key:
            return matched_key, labels
    return None, labels
```

---

#### 5. Create AWS Lambda Function from Container Image

```bash
aws lambda create-function \
    --function-name NutriVisionPredictor \
    --package-type Image \
    --code ImageUri=110359221458.dkr.ecr.ap-southeast-1.amazonaws.com/nutrivision-lambda:latest \
    --role arn:aws:iam::110359221458:role/service-role/NutriVisionPredictor-role-kzg0ihpv \
    --memory-size 512 \
    --timeout 30 \
    --environment "Variables={S3_BUCKET_NAME=fcaj-food-ai-storage-2026-sg-110359221458-ap-southeast-1-an}" \
    --region ap-southeast-1

aws lambda wait function-active-v2 --function-name NutriVisionPredictor --region ap-southeast-1
```

---

#### 6. Lambda Code Update Workflow (For Future Iterations)

Whenever **lambda_function.py** or **calorie_map.json** is modified, execute this fast update workflow:

```bash
cd backend/deploy

docker build --platform linux/amd64 --provenance=false -t nutrivision-lambda:latest .
docker tag nutrivision-lambda:latest 110359221458.dkr.ecr.ap-southeast-1.amazonaws.com/nutrivision-lambda:latest
docker push 110359221458.dkr.ecr.ap-southeast-1.amazonaws.com/nutrivision-lambda:latest

aws lambda update-function-code \
    --function-name NutriVisionPredictor \
    --image-uri 110359221458.dkr.ecr.ap-southeast-1.amazonaws.com/nutrivision-lambda:latest \
    --region ap-southeast-1

aws lambda wait function-updated --function-name NutriVisionPredictor --region ap-southeast-1
```

---

#### 7. Review Settings in AWS Lambda Console

Review your configuration on the [AWS Lambda Console](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/functions/NutriVisionPredictor):
- **Package type**: Image
- **Memory**: 512 MB
- **Timeout**: 30s
- **Environment variables**: `S3_BUCKET_NAME = fcaj-food-ai-storage-2026-sg-110359221458-ap-southeast-1-an`

![AWS Lambda Image URI Overview](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/lambda_image_overview.png)

![AWS Lambda Overview](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/lambda_step7_general_config.png)
