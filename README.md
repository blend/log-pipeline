# log-pipeline
Centralize all logs in a separate AWS account. See [blog post](https://blend.com/centralizing-logs-in-an-isolated-aws-account/).

## Deploying

1. Launch AWS Elasticsearch Domain.
`$ aws cloudformation create-stack --stack-name LogEsDomain --template-body file://es.yml --parameters ParameterKey=KibanaJumpboxPublicIpAddress,ParameterValue=x.x.x.x --profile isolated-account-admin`

2. Launch AWS Kinesis Firehose that delivers logs to Elasticsearch Domain in step 1.
`$ aws cloudformation create-stack --stack-name LogFirehose --template-body file://firehose.yml --parameters ParameterKey=ElasticsearchDomainArn,ParameterValue=<from_step_1> ParameterKey=FirehoseBackupS3KmsKeyArn,ParameterValue=<kms_key> --capabilities CAPABILITY_IAM --profile isolated-account-admin`

3. Launch log sink stack that accepts logs from another account and forwards them to the AWS Kinesis Firehose in step 2.
`$ aws cloudformation create-stack --stack-name LogSink --template-body file://cloudwatch_logs_sink.yml --parameters ParameterKey=LogSinkFirehoseDeliveryStream,ParameterValue=<from_step_2> ParameterKey=SubscriptionFilterAccountId,ParameterValue=111111111111 --capabilities CAPABILITY_IAM --profile isolated-account-admin`

4. Launch log source in each AWS account to send logs to the log sink in the isolated AWS account.
`$ aws cloudformation create-stack --stack-name LogSource --template-body file://cloudwatch_logs_source.yml --parameters ParameterKey=DestinationArn,ParameterValue=<from_step_3> --profile 111111111111-admin`

