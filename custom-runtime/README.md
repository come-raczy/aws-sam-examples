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
