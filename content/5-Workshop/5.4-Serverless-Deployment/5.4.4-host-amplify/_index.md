---
title: "Host Web UI on AWS Amplify"
date: 2026-08-04
weight: 4
chapter: false
pre: " <b> 5.4.4 </b> "
---

#### Host Frontend Web Application on AWS Amplify

After provisioning the API Gateway REST API, deploy the NutriVision web interface to 24/7 public availability with secure HTTPS using AWS Amplify Hosting.

#### 1. Prepare Frontend Source Code:
Ensure your web interface directory includes the necessary assets:
- index.html (Web structural layout)
- styles.css (Modern responsive styling)
- app.js (Inference submission script connecting to API Gateway)

Push the frontend source files to your GitHub repository

#### 2. Create AWS Amplify App via Console:
1. Sign in to the [AWS Amplify Console](https://ap-southeast-1.console.aws.amazon.com/amplify/home?region=ap-southeast-1).
2. Click Create new app ➔ Select source provider GitHub (or Deploy without Git provider).
3. Connect repository iumctocngan/NutriVision and select the main branch.

![AWS Amplify Select Repository and Branch](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/amplify_select_repo_branch.png)

4. On the App settings step:
   - App name: Enter `NutriVision`.
   - Frontend build command: Leave empty (pure static HTML/CSS/JS).

![AWS Amplify App Settings](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/amplify_step1_app_settings.png)

5. Advance to Review ➔ Verify configuration and click Save and deploy.

![AWS Amplify Review and Save and Deploy](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/amplify_step2_save_deploy.png)

#### 3. Access Public HTTPS Deployed Endpoint:
1. Once the automated build and deployment finishes (approximately 30 seconds), the NutriVision: Overview page displays the deployed status with the Visit deployed URL button.

![AWS Amplify Overview - Visit deployed URL](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/amplify_step3_visit_deployed_url.png)

2. Click Visit deployed URL to open the production web application:  
   `https://main.dnrnzxbbonuba.amplifyapp.com`

- Open this URL across any mobile or desktop browser.
- The web frontend is now fully wired to API Gateway for end-to-end testing!
