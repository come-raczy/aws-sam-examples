# AWS SAM Custom Runtime

The basic idea is to create a Docker image with a dedicated entry point.
A good starting point is the [AWS Lambda provided runtime
interface](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-api.html).

As we are deploying docker images, a good docker image to build upon is the [provided.al2023]
(https://aws.amazon.com/blogs/compute/introducing-the-amazon-linux-2023-runtime-for-aws-lambda/) image.

Note that this image only has "microdnf" and the list of available packages is limited to the [Amazon Linux 2023
RPM Packages](https://docs.aws.amazon.com/linux/al2023/release-notes/all-packages-AL2023.5.html).

## Build and deploy

Just make sure that `sam-cli` is installes, and that AWS credentials are properly set up.
The minimal deplotment is:

    sam build
    sam deploy --guided

## Test

Assuming that there isn't any othere Lambda function:

    FUNCTION_NAME=$(aws lambda list-functions | jq -r '.Functions[0].FunctionName')
    aws lambda invoke --function-name  $FUNCTION_NAME response.json

The function name can also retrieved with `sam-cli`:

    STACK_NAME=$(grep stack_name samconfig.toml | sed 's/^.*"\(.*\)"/\1/')
    sam list stack-outputs  --stack-name $STACK_NAME

## Minimal example

Just as the name says. A reasonable starting point for any lambda that runs a shell script or
a compiled binary.

## Environment variables

Based on the minimal example, this one adds the possibility to set and use environment variables.
See [Use Lambda environment variables to configure values in code]
(https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html).

### User defined environment variables

Environment variables can be set in several different ways:

- Console: from the [Function page](https://console.aws.amazon.com/lambda/home#/functions), choose the function and
  then the "Configuration" tab. There is a section called "Environment variables" where the variables can be created
  and modified. In the **Code** tab, there is an "Environment variables" section that allows listing the variables.
- AWS CLI: use the `--environment` option of the `aws lambda update-function-configuration` command. The syntax is
  in the form "Variables={BUCKET=amzn-s3-demo-bucket,KEY=file.txt}"
- AWS SAM: in the `template.yaml`, under the function specification, specify the variables under "Properties:" ->
  "Environment:" -> "Variables:". The syntax is in the form "BUCKET: amzn-s3-demo-bucket".

Note: when environment variables are secret and shouldn't be in the `template.yaml`, the only option seems to be using
the AWS CLI. In that case, it is important to set **all the environment variables**, otherwise the unset variables
will be removed:

    aws lambda update-function-configuration --function-name $FUNCTION_NAME \
        --environment "Variables={PUBLIC_VAR=some-value,SECRET_VAR=$SECRET_VAR}"

### Defined runtime environment variables

This is the list of environment variables that are defined by the runtime:

- `_HANDLER`: handler location - not relevant for docker images.
- `_X_AMZN_TRACE_ID`: AWS X-Ray tracing header. For custom runtime, it must be set using the `Lambda-Runtime-Trace-Id`
  response header from the [Next invocation]
  (https://docs.aws.amazon.com/lambda/latest/dg/runtimes-api.html#runtimes-api-next);
- `AWS_DEFAULT_REGION`: the default region where the Lambda function is executed;
- `AWS_REGION`: the region where the Lambda function is executed; overrides `AWS_DEFAULT_REGION` if defined.
- `AWS_EXECUTION_ENV`: The [runtime identifier](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html),
  prefixed by `AWS_Lambda_` (e.g. `AWS_Lambda_java8`)- not defined for OS-only runtimes.
- `AWS_LAMBDA_FUNCTION_NAME`: the name of the Lambda function
- `AWS_LAMBDA_FUNCTION_MEMORY_SIZE`: the amount of memory that is allocated for the function, in MB.
- `AWS_LAMBDA_FUNCTION_VERSION`: the version of the function that is executed.
- `AWS_LAMBDA_INITIALIZATION_TYPE`: the initialization type of the function (on-demand, provisioned-concurrency, or
  snap-start).
- `AWS_LAMBDA_LOG_GROUP_NAME`, `AWS_LAMBDA_LOG_STREAM_NAME`: the name of the CloudWatch Logs group and stream.
- `AWS_ACCESS_KEY`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` – The access keys obtained
  from the function's execution role.
- `**AWS_LAMBDA_RUNTIME_API**` – ([Custom runtime](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-custom.html))
  The host and port of the [runtime API](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-api.html).
- `LAMBDA_TASK_ROOT`: the path to the function code.
- `LAMBDA_RUNTIME_DIR`: the path to the runtime code.
