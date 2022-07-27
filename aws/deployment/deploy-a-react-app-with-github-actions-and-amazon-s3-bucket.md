#  Deploy a React app with GitHub Actions and Amazon S3 bucket

- [Deploy a React app with GitHub Actions and Amazon S3 bucket](#deploy-a-react-app-with-github-actions-and-amazon-s3-bucket)
  - [Create s3 bucket](#create-s3-bucket)
  - [Create IAM User](#create-iam-user)
  - [Create policy](#create-policy)
  - [Attach policy](#attach-policy)
  - [Create secrets in GitHub](#create-secrets-in-github)
  - [Create GitHub Actions](#create-github-actions)
  - [Create a CloudFront Distribution](#create-a-cloudfront-distribution)
  - [Create Error pages](#create-error-pages)
  - [Create invalidation](#create-invalidation)
  - [References](#references)

## Create s3 bucket

- Navigate to **https://console.aws.amazon.com/s3/**
- Choose **Create bucket**
- Bucket name: **example-ui**
- AWS Region: **Asia Pacific (Mumbai) ap-south-1**
- Click on **Add tag** under **Tags(0) - optional**
  - Key: **Project**
  - Value: **EXAMPLE**
- Click on **Create bucket** to create

## Create IAM User

- Navigate to **https://console.aws.amazon.com/iam/**
- In the navigation pane, choose **Users** and then choose **Add users**
- User name: **example-ui-github-actions**
- Select **Access key - Programmatic access** checkbox
- Choose **Next: Permissions**
- Choose **Next: Tags**
  - Key: **Project**
  - Value: **EXAMPLE**
- Choose **Next: Review**
- Choose **Create user**
- Click on **Download .csv** and save it safely, it has a secret keys.

## Create policy

- Navigate to **https://console.aws.amazon.com/iam/**
- In the navigation pane, choose **Policies** and then choose **Create policy**
- Click on **JSON**
- Copy and paste the below JSON code 
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Resource": [
                    "arn:aws:s3:::example-ui",
                    "arn:aws:s3:::example-ui/*"
                ],
                "Sid": "AllowS3SyncCommand",
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:ListBucket",
                    "s3:PutObject"
                ]
            }
        ]
    }
    ```
- Choose **Next: Tags** and click on **Add tag**
  - Key: **Project**
  - Value: **EXAMPLE**
- Choose **Next: Review**
  - Name: **deploy-example-ui-app**
  - Description: This policy is created to deploy the example-ui app from GitHub Actions.
- Choose **Create policy**

## Attach policy

- Navigate to **https://console.aws.amazon.com/iam/**
- In the navigation pane, choose **Users**
- Choose **example-ui-github-actions**
- Choose **Add permissions**
- Choose **Attach existing policies directly**
- Search for **deploy-example-ui-app** and choose 
- Choose **Next: Review**
- Click on **Add permissions** 

## Create secrets in GitHub

- Navigate to **https://github.com/** and login
- Choose the **example-ui** repository
- Click on **Settings**
- Click **Secrets** and choose **Actions**
- Click on **New repository secret** and create the secrets for the followings.
  - Name: **AWS_ACCESS_KEY_ID** Value: **access key id goes here** and click on **Add secret** repeat the same for other secrets.
  - Name: **AWS_SECRET_ACCESS_KEY** and Value: **secret key goes here**
  - Name: **AWS_REGION** and Value: **aws region goes here**
  - Name: **AWS_S3_BUCKET_NAME** and Value: **s3 bucket name goes here** 

## Create GitHub Actions

- Click on **Actions**
- Choose **set up a workflow yourself**
- Copy and paste the below code.

```yaml
name: 'Production deployment'

on:
 push:
   branches:
     - main
   paths-ignore:
     - '**.md'
 pull_request:
   paths-ignore:
     - '**.md'

jobs:
 Deploy:
 
  runs-on: ubuntu-latest
  
  strategy:
    matrix:
      node-version: [16.13.2]
      
  steps:
  - name: Checkout
    uses: actions/checkout@v3

  - name: Setup node
    uses: actions/setup-node@v3
    with:
     node-version: ${{ matrix.node-version }}
     # cache: npm

  - name: Install dependencies
    run: npm install --legacy-peer-deps

  - name: Build static file
    run: CI=false npm run build

  - name: Configure AWS Credentials to deploy
    uses: aws-actions/configure-aws-credentials@v1
    with:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      aws-region: ${{ secrets.AWS_REGION }}

  - name: Copy files to the s3 bucket with the AWS CLI
    run: |
      aws s3 sync ./build s3://${{ secrets.AWS_S3_BUCKET_NAME }}
```

- Click on **Start commit** at right side
- Choose **Commit new file**
- Click on **Actions** to check the status

## Create a CloudFront Distribution

- Navigate to **https://console.aws.amazon.com/cloudfront/**
- Choose **Create distribution**
- Origin domain: **example-ui-domain-name**
- S3 bucket access
  - Select **Yes use OAI (bucket can restrict access to only CloudFront)** 
  - Choose **Create new OAI**
  - Choose **Create**
- Bucket policy 
  - Select **Yes, update the bucket policy**
- Viewer protocol policy
  - Select **Redirect HTTP to HTTPS**
- Default root object - optional
  - Enter **index.html**
- Click on **Create distribution**
- Wait for some time till CloudFront Distribution creates successfully and upon success copy the **Distribution domain name**
- Open browser and paste the **Distribution domain name** to access website

## Create Error pages

**Create error page for `403 Forbidden`:**

- Choose **Create custom error response**
- HTTP error code: 
  - Select **403: Forbidden**
- Customize error response: 
  - Select **Yes** 
    - Response page path: Enter **/index.html**
- HTTP Response code: Select 
  - **200: OK**
- Choose **Create custom error response**

**Create error page for `404 Not Found`:**

- Choose **Create custom error response**
- HTTP error code: 
  - Select **403: Forbidden**
- Customize error response: 
  - Select **Yes** 
    - Response page path: Enter **/index.html**
- HTTP Response code: Select 
  - **200: OK**
- Choose **Create custom error response**

## Create invalidation

- Navigate to the Distribution
- Choose **Invalidations**
- Choose **Create invalidation**
- Add object paths: 
  - **/***
- Click on **Create invalidation**

## References

- ref: https://github.com/actions/setup-node/blob/main/docs/advanced-usage.md#caching-packages-data
- ref: https://github.com/aws-actions/configure-aws-credentials
- ref: https://github.com/actions/upload-artifact/actions/runs/101494005/workflow
