---
layout: post
title:  "AWS LocalStack"
date:   2025-04-20 21:16:02 +0200
categories: AWS LocalStack
---

### docker-compose file
```yaml
version: '3.8'
services:
  localstack:
    image: localstack/localstack
    container_name: localstack
    ports:
      - "4566:4566" # LocalStack Gateway
      - "4510-4559:4510-4559" # External services
    environment:
      - DOCKER_HOST=unix:///var/run/docker.sock
      - SERVICES=s3,sqs,lambda,dynamodb,apigateway,cloudwatch,cloudformation,iam
      - DEFAULT_REGION=us-east-1
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
#### create a bucket
```bash
aws --endpoint-url=http://localhost:4566 s3api create-bucket --bucket my-bucket
```
#### create a queue
```bash
aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name my-queue
```
#### create a lambda function
```bash
aws --endpoint-url=http://localhost:4566 lambda create-function \
  --function-name my-function \
  --runtime python3.8 \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```
#### create a dynamodb table
```bash
aws --endpoint-url=http://localhost:4566 dynamodb create-table \
  --table-name my-table \
  --attribute-definitions AttributeName=Id,AttributeType=S \
  --key-schema AttributeName=Id,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```
