aws logs put-metric-filter --log-group-name <ロググループ名> \
--filter-name CreateAccountAssignmentFilter \
--filter-pattern '{ $.eventName = "CreateAccountAssignment" }' \
--metric-transformations metricName=CreateAccountAssignmentCount,metricNamespace=CustomNamespace,metricValue=1

aws cloudwatch put-metric-alarm --alarm-name CreateAccountAssignmentAlarm \
--metric-name CreateAccountAssignmentCount \
--namespace CustomNamespace \
--statistic Sum \
--period 300 \
--threshold 1 \
--comparison-operator GreaterThanOrEqualToThreshold \
--evaluation-periods 1 \
--alarm-actions <SNSトピックARN>


provider "aws" {
  region = "us-east-1" # 必要に応じてリージョンを変更してください
}

resource "aws_cloudwatch_log_group" "example" {
  name = "/aws/lambda/example"
}

resource "aws_cloudwatch_log_metric_filter" "example" {
  name           = "CreateAccountAssignmentFilter"
  log_group_name = aws_cloudwatch_log_group.example.name
  pattern        = "{ $.eventName = \"CreateAccountAssignment\" }"

  metric_transformation {
    name      = "CreateAccountAssignmentCount"
    namespace = "CustomNamespace"
    value     = "1"
  }
}

resource "aws_sns_topic" "example" {
  name = "CreateAccountAssignmentAlertTopic"
}

resource "aws_sns_topic_subscription" "example" {
  topic_arn = aws_sns_topic.example.arn
  protocol  = "email"
  endpoint  = "your-email@example.com" # 通知を受け取るメールアドレス
}

resource "aws_cloudwatch_metric_alarm" "example" {
  alarm_name          = "CreateAccountAssignmentAlarm"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = "1"
  metric_name         = "CreateAccountAssignmentCount"
  namespace           = "CustomNamespace"
  period              = "300"
  statistic           = "Sum"
  threshold           = "1"
  alarm_description   = "This alarm monitors CreateAccountAssignment events"
  alarm_actions       = [aws_sns_topic.example.arn]
}
