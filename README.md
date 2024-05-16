# CI/CD Pipeline with Jenkins, SonarQube, Docker, ArgoCD and Kubernetes

This repository demonstrates a comprehensive Continuous Integration/Continuous Deployment (CI/CD) pipeline using Jenkins, SonarQube, Docker, Azure Container Registry, Argo CD, and Kubernetes. Below is a detailed explanation of each step in the workflow, accompanied by a visual diagram.

## Workflow Overview
The CI/CD pipeline in this repository is designed to automate the process of building, testing, and deploying applications. The workflow ensures that code changes are continuously integrated and deployed to a Kubernetes cluster with quality checks and monitoring.

![image](https://github.com/EzeChinedumUchenna/http-echo-project/assets/102483586/1ff6edc1-baff-47e3-8d35-b47cd484f701)

## Pipeline Steps
* __GitHub:__ Acts as the source control repository. Code changes are pushed to GitHub, which triggers the Jenkins build process.
* __Jenkins:__ Serves as the Continuous Integration server. It pulls the latest code from GitHub, runs unit tests to ensure code integrity, initiates a code quality analysis using SonarQube. If the code passes the quality checks, builds a Docker image.
* __SonarQube:__ Performs static code analysis.
Action: Analyzes the code for quality, bugs, vulnerabilities, and code smells. Sends the results back to Jenkins.
Docker:

Role: Containerizes the application.
Action: Jenkins builds a Docker image from the codebase.
Azure Container Registry:

Role: Stores Docker images.
Action: The Docker image is pushed to the Azure Container Registry for versioning and storage.
Argo CD:

Role: Manages continuous deployment.
Action: Pulls the Docker image from the Azure Container Registry and deploys it to the Kubernetes cluster.
Kubernetes:

Role: Orchestrates containerized applications.
Action: Runs the deployed Docker image, ensuring the application is up and running. Monitors the health and scaling of the application.
Email Notification:

Role: Alerts stakeholders.
Action: Sends notifications about the deployment status.




