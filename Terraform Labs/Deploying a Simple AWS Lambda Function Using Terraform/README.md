### Deploying a Simple AWS Lambda Function Using Terraform

In this lab, you will learn how to deploy a simple AWS Lambda function using Terraform. AWS Lambda is a serverless compute service that allows you to run code without provisioning or managing servers. You will deploy a basic Lambda function that returns a "Hello, World!" message when invoked.

## Objectives

1. **Understand AWS Lambda:** Learn the basics of AWS Lambda functions.
2. **Write a Simple Lambda Function:** Create a "Hello, World!" Lambda function using Node.js.
3. **Package the Lambda Function:** Package the Lambda function into a ZIP file.
4. **Deploy with Terraform:** Create and deploy the Lambda function using Terraform.
5. **Set Up IAM Role:** Configure an IAM role for the Lambda function with the necessary permissions.
6. **Invoke the Lambda Function:** Verify that the Lambda function works correctly by invoking it.

## What is an AWS Lambda Function?

AWS Lambda is a compute service that allows you to run code without managing servers. Simply write your code, upload it to Lambda, and the service takes care of scaling and execution. Lambda functions are often used in event-driven applications, such as responding to HTTP requests, processing S3 bucket files, or reacting to database changes.


## Project directory structure for the lab

```plaintext
terraform-lambda-lab/
├── function.zip
├── index.js
└── main.tf
```

- `index.js`: Contains the Node.js code for the AWS Lambda function.
- `function.zip`: The ZIP file containing the Lambda function code, created from `index.js`.
- `main.tf`: The Terraform configuration file that defines the AWS resources, including the Lambda function, IAM role, and permissions.

## Step 1: Writing the Lambda Function

### Create the Lambda Function

1. Create a directory for your Lambda function:

    ```bash
    mkdir terraform-lambda-lab
    cd terraform-lambda-lab
    ```

2. Inside this directory, create a file named `index.js` with the following content:

    ```javascript
    exports.handler = async (event) => {
        const response = {
            statusCode: 200,
            body: JSON.stringify('Hello, World!'),
        };
        return response;
    };
    ```

   This simple Node.js function returns a JSON response with the message "Hello, World!".

### Package the Lambda Function

Package the Lambda function into a ZIP file for deployment:

if `zip` is not installed in your machine ,Installed it by,

```bash
sudo apt install zip
```

Then apply this ,
```bash
zip function.zip index.js
```

This command creates a `function.zip` file containing your Lambda function code.


## Step 2: Setting Up Terraform

### Create the Terraform Configuration

Create a `main.tf` file with the following content:

```py
provider "aws" {
  region = "ap-southeast-1"
}

# IAM role for the Lambda function
resource "aws_iam_role" "lambda_role" {
  name = "lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Action    = "sts:AssumeRole",
      Effect    = "Allow",
      Principal = {
        Service = "lambda.amazonaws.com"
      },
    }],
  })

  tags = {
    Name = "lambda-role"
  }
}

# Attach the AWSLambdaBasicExecutionRole policy to the role
resource "aws_iam_role_policy_attachment" "lambda_policy" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

# Deploy the Lambda function
resource "aws_lambda_function" "hello_world" {
  function_name = "hello-world-function"
  role          = aws_iam_role.lambda_role.arn
  handler       = "index.handler"
  runtime       = "nodejs14.x"
  filename      = "${path.module}/function.zip"
  source_code_hash = filebase64sha256("${path.module}/function.zip")

  tags = {
    Name = "hello-world-function"
  }
}

# Permission to invoke the Lambda function
resource "aws_lambda_permission" "allow_invoke" {
  statement_id  = "AllowExecutionFromAny"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.hello_world.function_name
  principal     = "lambda.amazonaws.com"
}
```

### Explanation of `main.tf`

- **Provider Configuration:**
  - Specifies the AWS region where resources will be deployed.

- **IAM Role:**
  - **aws_iam_role.lambda_role:** Creates an IAM role that the Lambda function will assume. The role includes a policy allowing Lambda to assume the role.
  - **aws_iam_role_policy_attachment.lambda_policy:** Attaches the AWS-provided `AWSLambdaBasicExecutionRole` policy to the IAM role, granting permissions to write logs to CloudWatch.

- **Lambda Function:**
  - **aws_lambda_function.hello_world:** Deploys the Lambda function using the Node.js runtime. The function code is provided in the `function.zip` file, and the handler is set to `index.handler`.
  - **aws_lambda_permission.allow_invoke:** Grants permission for the Lambda function to be invoked by any AWS service.

## Step 3: Applying the Terraform Configuration

### Initialize Terraform

Initialize Terraform to set up the environment and download the necessary providers:

```bash
terraform init
```

### Apply the Configuration

Apply the Terraform configuration to deploy the Lambda function and related resources:

```bash
terraform apply
```

Type `yes` when prompted to confirm the creation of resources.

## Step 4: Invoking the Lambda Function

### Use the AWS CLI to Invoke the Function

Invoke the Lambda function using the AWS CLI:

```bash
aws lambda invoke \
    --function-name hello-world-function \
    --payload '{}' \
    response.json
```

This command invokes the Lambda function and saves the response to `response.json`.

### 4.2: Check the Response

Check the contents of `response.json`:

```bash
cat response.json
```

You should see the "Hello, World!" message in the response.


## Conclusion

In this lab, you created a simple "Hello, World!" AWS Lambda function using Node.js, packaged it into a ZIP file, and deployed it using Terraform. You also set up the necessary IAM role and permissions for the Lambda function and invoked it using the AWS CLI. This lab demonstrates the basics of deploying Lambda functions with Terraform, which you can build upon for more complex serverless applications.