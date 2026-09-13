---
title: "Step 3: CloudWatch Monitoring & SNS Alerts"
date: 2026-08-04
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Configure Amazon CloudWatch & Amazon SNS Alerts

In this section, you will configure Amazon CloudWatch to monitor the serverless backend running on AWS Lambda and integrate Amazon SNS to dispatch real-time incident alert notifications.

Amazon CloudWatch provides comprehensive monitoring capabilities by collecting operational metrics and runtime logs from AWS resources and evaluating them against user-defined thresholds. This enables administrators to detect runtime failures, observe AI inference latency, and respond promptly when anomalies occur.

In this project, CloudWatch is used to monitor execution failures (Errors) of the NutriVisionPredictor Lambda Function by creating a CloudWatch Alarm integrated with an Amazon SNS notification topic.

---

#### 1. Configure CloudWatch Logging

AWS Lambda natively integrates with Amazon CloudWatch to centralize runtime execution logs in the /aws/lambda/NutriVisionPredictor Log Group.

Tail recent Lambda execution logs via AWS CLI (filter within the last 7 days):
```bash
aws logs tail /aws/lambda/NutriVisionPredictor --region ap-southeast-1 --since 7d
```

Sample raw CloudWatch log output demonstrating model initialization, latency metrics, and Rekognition Fallback labels:
```text
[INFO] Using bundled food_model.onnx file.
[INFO] Loading calorie_map.json from local bundle.
[INFO] ONNX model initialized with 50 classes.
[INFO] Fallback labels: [('Food', 100.0), ('Meal', 100.0), ('Dish', 99.8), ('Cooking', 97.5), ('Bowl', 95.4), ('Soup', 55.3), ('Noodle', 52.8)]
REPORT RequestId: fe178352-12fe-4fdd-b73f-10ab4dfab1fb	Duration: 1359.21 ms	Billed Duration: 2482 ms	Memory Size: 512 MB	Max Memory Used: 233 MB	Init Duration: 1108.15 ms
```

Key telemetry parameters captured:
- Duration: Actual AI model execution latency in milliseconds.
- Billed Duration: Billed Lambda execution time.
- Memory Size & Max Memory Used: Allocated RAM (512 MB) and peak memory consumed.
- Application Logs: ONNX inference output status, Amazon Rekognition Fallback triggers, and Out-of-Distribution (OOD) image persistence logs.

---

#### 2. Configure Amazon SNS Notification Topic

To deliver immediate notifications to administrators when incidents occur, configure an Amazon SNS topic and subscribe an email address.

The Amazon SNS Topic is configured using the following settings:

| Property | Value |
|---|---|
| Topic type | Standard |
| Name | NutriVision-AlarmNotifications |
| Display name | NutriVision Alerts |
| Subscription Protocol | Email |
| Endpoint | chuasheef3@gmail.com |

Execute AWS CLI commands to create the topic and subscribe an email address:
```bash
aws sns create-topic --name NutriVision-AlarmNotifications --attributes DisplayName="NutriVision Alerts" --region ap-southeast-1

aws sns subscribe --topic-arn arn:aws:sns:ap-southeast-1:110359221458:NutriVision-AlarmNotifications --protocol email --notification-endpoint chuasheef3@gmail.com --region ap-southeast-1
```

> [!IMPORTANT]
> After creating the subscription, open your email inbox and click Confirm subscription in the message received from AWS Notifications to activate alert delivery.

![Amazon SNS Notification Email](/FCAJ-Project/images/5-Workshop/5.5-Monitoring-Alerts/sns_email_notification_confirmed.png)

---

#### 3. Create a CloudWatch Alarm

To continuously monitor the health of the serverless backend, a CloudWatch Alarm is configured for the NutriVisionPredictor Lambda Function.

The alarm evaluates the total number of execution failures (Sum of Errors) across a 1-minute period. If any unhandled error occurs (Errors >= 1), CloudWatch automatically changes the alarm state from OK to In alarm and triggers the SNS Topic to notify administrators immediately.

The CloudWatch Alarm is configured using the following settings:

| Property | Value |
|---|---|
| Alarm name | NutriVision-HighLambdaErrors |
| Alarm description | Alert when NutriVisionPredictor Lambda function encounters execution errors |
| Namespace | AWS/Lambda |
| Metric name | Errors |
| Statistic | Sum |
| Period | 1 minute |
| Threshold | >= 1 |
| Function name | NutriVisionPredictor |
| Alarm Action | Send notification to NutriVision-AlarmNotifications (SNS) |

Create the CloudWatch Alarm using the AWS CLI:
```bash
aws cloudwatch put-metric-alarm --alarm-name NutriVision-HighLambdaErrors --alarm-description "Alert when NutriVisionPredictor Lambda function encounters execution errors" --metric-name Errors --namespace AWS/Lambda --statistic Sum --dimensions Name=FunctionName,Value=NutriVisionPredictor --period 60 --evaluation-periods 1 --threshold 1 --comparison-operator GreaterThanOrEqualToThreshold --treat-missing-data notBreaching --alarm-actions arn:aws:sns:ap-southeast-1:110359221458:NutriVision-AlarmNotifications --region ap-southeast-1
```

---

#### 4. Verify the Alarm Status

After the alarm is created, Amazon CloudWatch continuously tracks the health and execution state of the NutriVisionPredictor Lambda function.

![CloudWatch Alarm OK Status Dashboard](/FCAJ-Project/images/5-Workshop/5.5-Monitoring-Alerts/cloudwatch_alarm_ok_status.png)

- Normal state (OK): The alarm stays green in the OK state while the function processes requests successfully without runtime exceptions (Errors < 1).
- Incident state (In alarm): If execution failures reach or exceed the threshold (Errors >= 1), CloudWatch automatically transitions the state to In alarm (red) and sends an instant email notification via Amazon SNS.

The alarm detail page displays comprehensive operational telemetry:
- Current alarm state (OK / In alarm).
- Errors metric graph over time.
- Threshold configuration and evaluation period.
- Monitored Lambda function name and namespace.
- Triggered Amazon SNS action history.

This information enables administrators to understand the current operating condition of the deployed serverless application and quickly pinpoint root causes.
