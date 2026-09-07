---
title: "Tạo Amazon API Gateway REST API"
date: 2026-08-04
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

#### Các bước khởi tạo Amazon API Gateway REST API qua AWS Console

Khởi tạo cổng giao tiếp REST API NutriVisionRestApi để tiếp nhận request HTTPS từ giao diện Web, xử lý cấu hình CORS, thiết lập Rate Limiting chặn Spam và định tuyến dữ liệu trực tiếp đến hàm AWS Lambda:

1. **Truy cập Amazon API Gateway Console:**
   - Đăng nhập vào [Amazon API Gateway Console](https://ap-southeast-1.console.aws.amazon.com/apigateway/main/apis?region=ap-southeast-1).
   - Tại màn hình danh sách APIs, nhấp nút **Create API**.

![Amazon API Gateway Overview - Create API](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step1_dashboard.png)

2. **Chọn loại API (REST API):**
   - Tìm đến thẻ dịch vụ **REST API** *(không chọn REST API Private và không chọn HTTP API)*.
   - Nhấp nút **Build** trên thẻ **REST API**.

![Select REST API - Click Build](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step2_select_rest.png)

3. **Cấu hình thông tin API:**
   - **Create new API**: Chọn **New API**.
   - **API name**: Nhập `NutriVisionRestApi`.
   - **Description**: Nhập `REST API cho NutriVision Predictor`.

![Configure API details NutriVisionRestApi](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step3_api_details.png)

4. **Cấu hình Endpoint Type & Khởi tạo API:**
   - **API endpoint type**: Chọn **Regional** (để tối ưu hóa độ trễ xử lý trong khu vực Singapore).
   - **Security policy**: Giữ mặc định.
   - Nhấp nút **Create API** ở góc dưới cùng bên phải.

![Configure API endpoint type Regional - Click Create API](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step4_endpoint_regional.png)

5. **Xác nhận khởi tạo thành công & Bấm Create resource:**
   - Màn hình hiển thị thông báo tạo REST API thành công.
   - Nhấp nút **Create resource** ở khung điều khiển phía bên trái.

![API Created Successfully - Click Create resource](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step5_create_resource_btn.png)

6. **Tạo Resource /predict:**
   - **Resource name**: Nhập `predict`.
   - Tích chọn **CORS (Cross Origin Resource Sharing)** để tự động sinh method OPTIONS.
   - Nhấp nút **Create resource**.

![Create Resource details predict](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step6_create_predict_resource.png)

7. **Xác nhận tạo Resource thành công & Bấm Create method:**
   - Nhấp chọn Resource **/predict** vừa tạo.
   - Nhấp nút **Create method** ở khung bên phải.

![Resource /predict Created Successfully - Click Create method](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step7_resource_created_click_create_method.png)

8. **Cấu hình Method Type POST & Integration Lambda Function:**
   - **Method type**: Chọn **POST**.
   - **Integration type**: Chọn **Lambda function**.

![Method details POST & Integration Lambda function](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step8_create_method_post_lambda.png)

9. **Đấu nối Hàm Lambda NutriVisionPredictor & Cấp quyền gọi:**
   - Bật công tắc **Lambda proxy integration**.
   - **Lambda function**: Tìm và chọn hàm **NutriVisionPredictor**.
   - Nhấp nút **Create method** ở góc dưới.

![Configure Lambda Proxy Integration & Select NutriVisionPredictor Function](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step9_select_lambda_function.png)

10. **Xác nhận cấu hình Methods OPTIONS & POST ➔ Bấm Enable CORS:**
    - Khung Methods hiển thị đầy đủ 2 method: **OPTIONS** (Preflight check) và **POST** (Lambda Integration).
    - Nhấp nút **Enable CORS** ở góc trên bên phải.

![Methods OPTIONS & POST Configured - Click Enable CORS](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step10_methods_options_post_cors.png)

11. **Tích chọn thông số CORS & Lưu cấu hình:**
    - Tại trang CORS settings: Tích chọn **Default 4XX**, **Default 5XX**, **OPTIONS**, **POST**.
    - Giữ nguyên **Access-Control-Allow-Headers** và **Access-Control-Allow-Origin: '*'**.
    - Nhấp nút **Save** ở góc dưới cùng.

![CORS Settings Checked Default 4XX 5XX OPTIONS POST - Click Save](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step11_cors_settings_checked.png)

12. **Mở cửa sổ Deploy API:**
    - Quay lại màn hình Resources tổng quan, nhấp nút **Deploy API** ở góc trên cùng bên phải.

![Resources Overview - Click Deploy API](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step12_click_deploy_api.png)

13. **Khai báo Stage prod & Phát hành API:**
    - **Stage**: Chọn **New Stage**.
    - **Stage name**: Nhập `prod`.
    - Nhấp nút **Deploy**.

![Deploy API Dialog - Stage prod - Click Deploy](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step13_deploy_modal_prod_stage.png)

14. **Mở trang điều chỉnh Cấu hình Stage prod:**
    - Sau khi Deploy, hệ thống tự động chuyển sang trang Stages.
    - Tại mục **Stage details** của stage **prod**, nhấp nút **Edit** ở góc trên bên phải.

![Stage Details prod - Click Edit](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step14_stage_click_edit.png)

15. **Bật Throttling Chống Spam (Rate: 20, Burst: 40):**
    - Tại mục **Throttling settings**: Bật công tắc **Throttling**.
    - **Rate**: Nhập `20` (requests per second).
    - **Burst**: Nhập `40` (requests).

![Edit Stage - Throttling settings Rate 20 & Burst 40](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step15_throttling_rate_20_burst_40.png)

16. **Xác nhận Thay đổi & Nhấp Save changes:**
    - Giao diện chuyển sang màn hình **Review changes to 'prod'** hiển thị bảng tóm tắt: Throttle rate 20 req/s, Throttle burst 40 req, Throttling Active.
    - Nhấp nút **Save changes** ở góc dưới cùng bên phải để áp dụng ngay lập tức.

![Review changes to prod - Click Save changes](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step16_review_save_changes.png)

17. **Trích xuất Invoke URL & Hoàn tất:**
    - Màn hình hiển thị thông báo phát hành thành công cho Stage prod.
    - Vào mục **Stages ➔ prod ➔ /predict ➔ POST** để sao chép đường dẫn **Invoke URL**:  
      `https://juv9jodvpc.execute-api.ap-southeast-1.amazonaws.com/prod/predict`

![Stages prod Active - Copy Invoke URL](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step14_stage_prod_invoke_url_copied.png)
