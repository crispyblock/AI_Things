# Diagram

mermaid```
flowchart TD

    SNS[SNS Topic\napp-notification-topic] -->|Triggers| LAMBDA[Lambda Function\nsns-trigger-lambda]

    LAMBDA --> LOGS[CloudWatch Logs]

    subgraph IAM
        ROLE[Lambda Execution Role]
        PERM[SNS Invoke Permission]
    end

    ROLE --> LAMBDA
    SNS --> PERM
```

```hcl
########################################
# SNS Topic
########################################
resource "aws_sns_topic" "notifications" {
  name = "app-notification-topic"
}

########################################
# IAM Role for Lambda
########################################
resource "aws_iam_role" "lambda_role" {
  name = "sns_lambda_execution_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_basic" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

########################################
# Lambda Function
########################################
resource "aws_lambda_function" "sns_handler" {
  function_name = "sns-trigger-lambda"
  handler       = "index.handler"
  runtime       = "nodejs18.x"
  filename      = "lambda.zip"
  role          = aws_iam_role.lambda_role.arn
}

########################################
# SNS → Lambda Subscription
########################################
resource "aws_sns_topic_subscription" "lambda_subscription" {
  topic_arn = aws_sns_topic.notifications.arn
  protocol  = "lambda"
  endpoint  = aws_lambda_function.sns_handler.arn
}

########################################
# Permission: Allow SNS to invoke Lambda
########################################
resource "aws_lambda_permission" "allow_sns" {
  statement_id  = "AllowExecutionFromSNS"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.sns_handler.function_name
  principal     = "sns.amazonaws.com"
  source_arn    = aws_sns_topic.notifications.arn
}
```
