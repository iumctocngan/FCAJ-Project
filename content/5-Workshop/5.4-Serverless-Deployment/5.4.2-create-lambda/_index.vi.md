---
title: "Triển khai AWS Lambda Container Image"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

#### Quy trình Đóng gói Docker Container Image & Triển khai AWS Lambda qua AWS CLI

Mô hình AI nhận diện món ăn yêu cầu các thư viện C++ native gồm onnxruntime, numpy và Pillow. Để đảm bảo mô hình hoạt động ổn định và nhất quán trên môi trường đám mây, toàn bộ mã nguồn xử lý và mô hình được đóng gói thành **Docker Container Image** và triển khai lên **AWS Lambda (PackageType Image)** bằng **AWS CLI**.

---

#### 1. Chuẩn bị Thư mục Build & Tệp Dockerfile

Đảm bảo ứng dụng **Docker Desktop** đã được khởi chạy trên máy tính.

Kiểm tra thư mục [backend/deploy/](https://github.com/iumctocngan/NutriVision/tree/main/backend/deploy) có đầy đủ các tệp cần thiết:
- **Dockerfile**
- **requirements.txt**
- [lambda_function.py](https://github.com/iumctocngan/NutriVision/blob/main/backend/deploy/lambda_function.py)
- **food_model.onnx** (15.5 MB)
- **calorie_map.json**

Nội dung tệp **Dockerfile**:
```dockerfile
FROM public.ecr.aws/lambda/python:3.12

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY lambda_function.py food_model.onnx calorie_map.json ./

CMD [ "lambda_function.lambda_handler" ]
```

> [!TIP] Tối ưu Cold Start:
> Đóng gói trực tiếp file mô hình **food_model.onnx** và **calorie_map.json** vào container image giúp Lambda nạp mô hình từ bộ nhớ cục bộ chỉ trong khoảng 1 giây, giảm thiểu độ trễ mạng so với việc tải liên tục từ S3.

---

#### 2. Khởi tạo Amazon ECR Repository & Đăng nhập

```bash
aws ecr create-repository --repository-name nutrivision-lambda --region ap-southeast-1

aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 110359221458.dkr.ecr.ap-southeast-1.amazonaws.com
```

---

#### 3. Build Docker Image & Push lên Amazon ECR

Chuyển vào thư mục `backend/deploy` và thực hiện build:

```bash
cd backend/deploy

docker build --platform linux/amd64 --provenance=false -t nutrivision-lambda:latest .

docker tag nutrivision-lambda:latest 110359221458.dkr.ecr.ap-southeast-1.amazonaws.com/nutrivision-lambda:latest

docker push 110359221458.dkr.ecr.ap-southeast-1.amazonaws.com/nutrivision-lambda:latest
```

![Amazon ECR Image Pushed](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/lambda_ecr_image_pushed.png)

---

#### 4. Cấu hình Quyền & Tích hợp Amazon Rekognition Fallback

Amazon Rekognition là dịch vụ AI không máy chủ (Serverless Managed API), hoạt động theo cơ chế On-Demand API nên **không cần khởi tạo máy chủ hay cấu hình phức tạp trên Console**.

1. **Gán quyền IAM Role cho Lambda gọi dịch vụ Rekognition và S3:**
```bash
aws iam attach-role-policy --role-name NutriVisionPredictor-role-kzg0ihpv --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam attach-role-policy --role-name NutriVisionPredictor-role-kzg0ihpv --policy-arn arn:aws:iam::aws:policy/AmazonRekognitionFullAccess
```

2. **Đoạn mã tích hợp Amazon Rekognition trong [lambda_function.py](https://github.com/iumctocngan/NutriVision/blob/main/backend/deploy/lambda_function.py):**
Mã nguồn Lambda sử dụng AWS Boto3 SDK để tự động kích hoạt Rekognition khi mô hình chính có độ tin cậy thấp:
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

#### 5. Khởi tạo Hàm AWS Lambda từ Container Image

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

#### 6. Quy trình Cập nhật Code Lambda (Khi có chỉnh sửa sau này)

Khi bạn chỉnh sửa mã nguồn **lambda_function.py** hoặc cập nhật bảng calo, thực hiện quy trình cập nhật nhanh:

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

#### 7. Rà soát Thông số trên AWS Lambda Console

Kiểm tra lại cấu hình trên [AWS Lambda Console](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/functions/NutriVisionPredictor):
- **Package type**: Image
- **Memory**: 512 MB
- **Timeout**: 30s
- **Environment variables**: `S3_BUCKET_NAME = fcaj-food-ai-storage-2026-sg-110359221458-ap-southeast-1-an`

![AWS Lambda Image URI Overview](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/lambda_image_overview.png)

![AWS Lambda Overview](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/lambda_step7_general_config.png)
