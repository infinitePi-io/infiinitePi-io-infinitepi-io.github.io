---
title : 'Lambda Function Url'
author: "Satish tripathi"
date : 2024-03-28T23:57:47+05:30
linktitle: "Lambda Function Url"
---
## [What is function url?](https://docs.aws.amazon.com/lambda/latest/dg/urls-invocation.html#urls-invocation-basics)

AWS Lambda Function URLs are unique HTTP endpoints that we can create using AWS Console, SDK or any other IaC tool. These URLs are used to trigger the Lambda function, and they can be integrated with a variety of workloads. Function URLs are dual stack-enabled, supporting IPv4 and IPv6. After setting up a function URL for the lambda function, it can be invoked through its HTTP(S) endpoint via a web browser, curl, Postman, or any HTTP client.

## **[Authorization](https://docs.aws.amazon.com/lambda/latest/dg/urls-auth.html)**

The **AuthType** parameter determines how Lambda authorizes requests to your function URL. Depending on the configuration you choose, resource-based policies can enable other or same AWS account roles to invoke the Lambda function.

* `AWS_IAM` – Lambda uses AWS Identity and Access Management (IAM) to authenticate and authorize requests based on the IAM principal's identity policy and the function's resource-based policy.
* `NONE` – Lambda doesn't perform any authentication before invoking  function. However, the function resource-based policy is always in effect and must grant public access before your function URL can receive requests.

**The recommanded way is to use Auth Type as IAM until there is no real requirment to expose the function url**

### [Resource based policy example for auth type as NONE:](https://docs.aws.amazon.com/lambda/latest/dg/urls-auth.html#urls-auth-none)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "StatementId": "FunctionURLAllowPublicAccess",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "lambda:InvokeFunctionUrl",
      "Resource": "arn:aws:lambda:${region}:xxxx:function:${function_name}",
      "Condition": {
        "StringEquals": {
          "lambda:FunctionUrlAuthType": "NONE"
        }
      }
    }
  ]
}
```

### [Resource based policy example for auth type as IAM:](https://docs.aws.amazon.com/lambda/latest/dg/urls-auth.html#urls-auth-iam)

**Lambda is in the same account as role:**

Invoke lambda function using example role.

```json
{
  "Version": "2012-10-17",
  "Id": "default",
  "Statement": [
    {
      "Sid": "FunctionURLWithAuthTypeIAM",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::xxxx:role/example"
      },
      "Action": "lambda:InvokeFunctionUrl",
      "Resource": "arn:aws:lambda:${region}:xxxx:function:${function_name}",
      "Condition": {
        "StringEquals": {
          "lambda:FunctionUrlAuthType": "AWS_IAM"
        }
      }
    }
  ]
}
```

****Lambda function is in a different account than the role:****

Invoke lambda function using example role in the different account.

```json
{
  "Version": "2012-10-17",
  "Id": "default",
  "Statement": [
    {
      "Sid": "CrossAccountFunctionURLWithAuthTypeIAM",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::yyyy:role/example"
      },
      "Action": "lambda:InvokeFunctionUrl",
      "Resource": "arn:aws:lambda:${region}:xxxx:function:${function_name}",
      "Condition": {
        "StringEquals": {
          "lambda:FunctionUrlAuthType": "AWS_IAM"
        }
      }
    }
  ]

```

## Ways to Invoke lambda function:

Using [aws4](https://www.npmjs.com/package/aws4) package:

```javascript
const aws4 = require('aws4');
const https = require('https');

// Define your AWS credentials and the function URL
const credentials = {
  accessKeyId: '',
  secretAccessKey: '',
  sessionToken: ''
};

const functionUrl = 'lambda-function-url-endpoint'
const region = 'lambda-function-deployed-region'

// Define the request options
const requestOptions = {
  region: '${region}',
  host: functionUrl,
  path: '/',
  service: 'lambda',
  method: 'POST',
  body: '{"Hello": "Invoking the lambda","Using": "Function Url"}',
  headers: {
    'Content-Type': 'application/json',
  },
};

// Sign the request using aws4
const signedRequest = aws4.sign(requestOptions, credentials);
function request(signedRequest) { 
    https.request(
        signedRequest, 
        function(res) { 
            res.pipe(process.stdout) 
        }).end(signedRequest.body || '') 
}
request(signedRequest)
```

**Using curl command:**

```bash
curl -X POST --user "$AWS_ACCESS_KEY_ID:$AWS_SECRET_ACCESS_KEY" "$FUNCTION_URL" \
        -H 'content-type: application/json' -H "x-amz-security-token: $AWS_SESSION_TOKEN" \
        -d '{"Hello": "Invoking the lambda","Using": "Function Url"}' \
        --aws-sigv4 "aws:amz:${region}:lambda"
```

## Difference Between AWS Lambda Function URLs & Amazon API Gateway

| Resource                    | AWS Lambda Function URLs                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Amazon API Gateway API                                                                                                                                                                                                                                                                                                      |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| API Type Support            | HTTP                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | API Gateway + Lambda                                                                                                                                                                                                                                                                                                        |
| AuthType                    | IAM, None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | HTTP, REST, Websocket                                                                                                                                                                                                                                                                                                       |
| Custom Domain Support       | No (Required to use CloudFront)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | API Key, IAM, Cognito, Lambda                                                                                                                                                                                                                                                                                               |
| Manage API Key              | No                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Yes                                                                                                                                                                                                                                                                                                                         |
| Caching                     | No                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Yes                                                                                                                                                                                                                                                                                                                         |
| CORS                        | Yes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Yes                                                                                                                                                                                                                                                                                                                         |
| Access Logs in CloudWatch   | No                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Yes                                                                                                                                                                                                                                                                                                                         |
| CloudWatch Metrics          | Yes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Yes                                                                                                                                                                                                                                                                                                                         |
| Response Timeout            | 15 minutes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | 29 seconds                                                                                                                                                                                                                                                                                                                  |
| Usage Plans                 | No                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Yes                                                                                                                                                                                                                                                                                                                         |
| Proxy to other AWS services | No                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Yes                                                                                                                                                                                                                                                                                                                         |
| Integration with AWS WAF    | No (Possible through CloudFront)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | Yes                                                                                                                                                                                                                                                                                                                         |
| Request and Response format | API Gateway payload format 2.0                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | API Gateway payload format 2.0                                                                                                                                                                                                                                                                                              |
| Price                       | Apply price of Lambda only                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | Yes                                                                                                                                                                                                                                                                                                                         |
| Endpoint URL Format         | `https://<url-id>.lambda-url.<region>.on.aws`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | `"https://{restapi_id}.execute-api.{region}.amazonaws.com/{stage_name}/"`                                                                                                                                                                                                                                                 |
| Use Case                    | Function URLs are best for use cases where you must implement a single-function microservice with a<br />public endpoint that doesn’t require the advanced functionality of API Gateway,such as request validation, <br />throttling, custom authorizers, custom domain names, usage plans,or caching. For example, when you are <br />implementing webhook handlers, form validators,mobile payment processing, advertisement placement, <br />machine learning inference, and so on.It is also the simplest way to invoke your Lambda functions during <br />research and development without leaving the Lambda console or integrating additional services. | SaaS applications where need to track limit usage using API Gateway usage plans.<br />Real-time applications using WebSockets. Cases where API response time within 29 seconds.<br />Cases where required advance Authorization using Cognito, Throttling, Caching, <br />Service proxy to other AWS services, etc features |
