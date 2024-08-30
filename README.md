# VE3-assingment
# CI/CD Pipeline for Dockerized Web Application

## Overview

This repository contains a CI/CD pipeline using GitHub Actions to deploy a Java web application packaged in a `.war` file to AWS Elastic Container Service (ECS) using Docker. The pipeline automates the process of building Docker images, pushing them to Amazon Elastic Container Registry (ECR), deploying them to ECS, and running integration tests.

## Prerequisites

Before you can use this pipeline, ensure you have the following:

- **GitHub Repository**: A repository containing your application code and Dockerfile.
- **AWS Account**: Access to an AWS account with permissions to manage ECS, ECR, and IAM roles.
- **Docker**: Basic understanding of Docker and Dockerfile.
- **AWS CLI**: Optional, for manual verification and testing.

## Setup

### 1. **AWS Setup**

1. **Amazon ECR**:
   - Create an ECR repository to store your Docker images.
   - Note the repository name.

2. **Amazon ECS**:
   - Set up an ECS cluster and service to deploy your application.
   - Note the cluster name and service name.

3. **IAM Roles**:
   - Ensure your GitHub Actions runner has permissions to interact with ECR and ECS.

### 2. **GitHub Secrets**

Store the following secrets in your GitHub repository:

1. **Navigate to GitHub Secrets:**
   - Go to `Settings` > `Secrets and variables` > `Actions`.

2. **Add Secrets:**
   - **`AWS_ACCESS_KEY_ID`**: Your AWS access key ID.
   - **`AWS_SECRET_ACCESS_KEY`**: Your AWS secret access key.
   - **`AWS_REGION`**: The AWS region where your resources are located (e.g., `us-east-1`).
   - **`ECR_REPOSITORY`**: The name of your ECR repository.
   - **`AWS_ACCOUNT_ID`**: Your AWS account ID.
   - **`ECS_CLUSTER`**: The name of your ECS cluster.
   - **`ECS_SERVICE`**: The name of your ECS service.
3. **Usage**

    **`Commit and Push`**:
        Commit your changes and push them to the main branch of your GitHub repository.

    **`Monitor Workflow`**:
        Navigate to the Actions tab in your GitHub repository to monitor the pipeline’s progress.

    **`Troubleshooting`**

    Pipeline Failures: Check the logs in the GitHub Actions tab for error details.
    Deployment Issues: Verify ECS configurations, including task definitions and service settings.
    Integration Tests: Ensure your test endpoints are correctly configured and accessible.

## Snapshots


### 4. **Configure Workflow File**

Ensure you have the GitHub Actions workflow file located at `.github/workflows/ci-cd.yml` with the following content:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Cache Docker layers
        uses: actions/cache@v3
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-docker-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-docker-

      - name: Build Docker image
        run: |
          docker buildx build --platform linux/amd64 -t ${{ secrets.ECR_REPOSITORY }}:${{ github.sha }} .

      - name: Log in to Amazon ECR
        uses: aws-actions/amazon-ecr-login@v1
        with:
          region: ${{ secrets.AWS_REGION }}

      - name: Push Docker image to ECR
        run: |
          docker tag ${{ secrets.ECR_REPOSITORY }}:${{ github.sha }} ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}:${{ github.sha }}
          docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}:${{ github.sha }}

      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ecs-task-definition.json
          service: ${{ secrets.ECS_SERVICE }}
          cluster: ${{ secrets.ECS_CLUSTER }}
          wait-for-service-stability: true

      - name: Integration Tests
        run: |
          curl -f http://your-ecs-service-url || exit 1

      - name: Rollback on failure
        if: failure()
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ecs-task-definition.json
          service: ${{ secrets.ECS_SERVICE }}
          cluster: ${{ secrets.ECS_CLUSTER }}
          wait-for-service-stability: true
          action: "rollback"


**


