---
title: "Create Amazon API Gateway REST API"
date: 2026-08-04
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

#### Step-by-Step Amazon API Gateway REST API Setup via AWS Console

Initialize a REST API gateway named NutriVisionRestApi to receive HTTPS requests from the web UI, handle CORS, configure throttling rate limits against spam, and route traffic directly to AWS Lambda:

1. Access Amazon API Gateway Console:
   - Log in to the [Amazon API Gateway Console](https://ap-southeast-1.console.aws.amazon.com/apigateway/main/apis?region=ap-southeast-1).
   - On the APIs list dashboard, click Create API.

![Amazon API Gateway Overview - Create API](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step1_dashboard.png)

2. Select API Type (REST API):
   - Locate the REST API card (do not select REST API Private or HTTP API).
   - Click Build on the REST API card.

![Select REST API - Click Build](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step2_select_rest.png)

3. Configure API Details:
   - Create new API: Select New API.
   - API name: Enter `NutriVisionRestApi`.
   - Description: Enter `REST API for NutriVision Predictor`.

![Configure API details NutriVisionRestApi](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step3_api_details.png)

4. Configure Endpoint Type & Create API:
   - API endpoint type: Select Regional (optimized for Singapore region latency).
   - Security policy: Keep default.
   - Click Create API at the bottom right.

![Configure API endpoint type Regional - Click Create API](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step4_endpoint_regional.png)

5. Verify API Creation & Click Create Resource:
   - The screen confirms REST API creation.
   - Click Create resource on the left navigation pane.

![API Created Successfully - Click Create resource](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step5_create_resource_btn.png)

6. Create Resource /predict:
   - Resource name: Enter `predict`.
   - Check CORS (Cross Origin Resource Sharing) to generate the OPTIONS method.
   - Click Create resource.

![Create Resource details predict](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step6_create_predict_resource.png)

7. Verify Resource Creation & Click Create Method:
   - Select the newly created /predict resource.
   - Click Create method on the right pane.

![Resource /predict Created Successfully - Click Create method](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step7_resource_created_click_create_method.png)

8. Configure Method Type POST & Lambda Integration:
   - Method type: Select POST.
   - Integration type: Select Lambda function.

![Method details POST & Integration Lambda function](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step8_create_method_post_lambda.png)

9. Connect Lambda Function NutriVisionPredictor:
   - Turn on Lambda proxy integration.
   - Lambda function: Search and select NutriVisionPredictor.
   - Click Create method at the bottom.

![Configure Lambda Proxy Integration & Select NutriVisionPredictor Function](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step9_select_lambda_function.png)

10. Enable CORS for Methods:
    - The Methods pane displays both OPTIONS and POST methods.
    - Click Enable CORS at the top right.

![Methods OPTIONS & POST Configured - Click Enable CORS](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step10_methods_options_post_cors.png)

11. Select CORS Parameters & Save:
    - On the CORS settings page, check Default 4XX, Default 5XX, OPTIONS, and POST.
    - Leave Access-Control-Allow-Headers and Access-Control-Allow-Origin: '*' as default.
    - Click Save at the bottom.

![CORS Settings Checked Default 4XX 5XX OPTIONS POST - Click Save](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step11_cors_settings_checked.png)

12. Open Deploy API Dialog:
    - Return to the Resources overview and click Deploy API at the top right.

![Resources Overview - Click Deploy API](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step12_click_deploy_api.png)

13. Configure Stage prod & Deploy:
    - Stage: Select New Stage.
    - Stage name: Enter `prod`.
    - Click Deploy.

![Deploy API Dialog - Stage prod - Click Deploy](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step13_deploy_modal_prod_stage.png)

14. Edit Stage prod Settings:
    - After deployment, navigate to Stages.
    - Under Stage details for prod, click Edit at the top right.

![Stage Details prod - Click Edit](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step14_stage_click_edit.png)

15. Enable Throttling Settings (Rate: 20, Burst: 40):
    - Under Throttling settings, enable Throttling.
    - Rate: Enter `20` (requests per second).
    - Burst: Enter `40` (requests).

![Edit Stage - Throttling settings Rate 20 & Burst 40](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step15_throttling_rate_20_burst_40.png)

16. Confirm Changes & Save:
    - Review the configuration summary: Throttle rate 20 req/s, Throttle burst 40 req, Throttling Active.
    - Click Save changes at the bottom right.

![Review changes to prod - Click Save changes](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step16_review_save_changes.png)

17. Retrieve Invoke URL:
    - Go to Stages ➔ prod ➔ /predict ➔ POST to get the Invoke URL:  
      `https://juv9jodvpc.execute-api.ap-southeast-1.amazonaws.com/prod/predict`

![Stages prod Active - Copy Invoke URL](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/apigw_step14_stage_prod_invoke_url_copied.png)
