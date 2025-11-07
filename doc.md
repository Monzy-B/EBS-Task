### DevOps, Cloud Native eCommerce Platform

## Objective
The objective is to design and implement a cloud-native, event driven eCommerce platform focused on scalability, resilency, and automation. The platform follows a microservices architecture and leverages Kubernetes, Helm, GitHub Actions, AWS EKS, and AWS ECR for deployment and orchestration. The entire setup emphasizes observability, continuous delivery, and fault tolerance.


## Core Services
The application consists of four core microservices and one message broker

1. Order Service
2. Inventory Service
3. Payment Service
4. Notification Service
5. Message Broker (RabbitMQ)

## Architecture and Infrastructure Design

The platform is deployed on Amazon Elastic Kubernetes Service (EKS) for high availability, scalability, and automatic container orchestration.

## Infrastructure Components
1. Kubernetes Cluster: Hosts all microsrvices, message broker and other supporting tools like Prometheus, Grafana, ArgoCD.
2. AWS ECR: Serves as a centralized Docker image repository for all service images.
3. Ingress Controller: Manages external access to services. TLS termination is handled via cert-manager and AWS Certificate Manager.
4. RabbitMQ Statefulset: This provides durable message queueing and ensures persistence through kubernetes persistent volumes.
5. Databaase (AWS Managed RDS)
6. ArgoCD: Automates GitOps-based deployment and synchronization with the Helm Chart Repository.
7. Prometheus and Grafana: Provides Monitoring and Metrics Visualization.
8. Alertmanager and Slack Integration: Notify the DevOps team of incident such as service downtime or queue overloads.

## Architecture Flow
- Developers Push code changes to Github
- Github Actions runs automated tests, static analysis, builds image and pushes image to AWS ECR.
- Depeneding on the selected CD methode the pipeline either updates the Helm Chart Values so that ArgoCD automatically applies the new image or Executed a helm upgrade command directly.
- Prometheus continuously scrapes metrics from all services and RabbitMQ exporters, while Grafana visualizes those metrics.
- Alerts are generated in Prometheus and routed via Alertmanager to a dedicated slack channel.
- Logs from all containers are aggregated through Loki for centralized analysis


## Deployment Strategy (Blue-Green Deployment)

The Blue Green deployment model is what I would use to achieve zero-downtime updates.

Two environments are maintained simultaneously. The active handles live production traffic while the other serves as a staging target for new releases.

# Deployment of Green Environment:
A new version of the application is deployed to the inactive (Green) environment.

# Validation and Testing:
Automated smoke tests, health probes, and integration checks validate the Green environment.

# Traffic Switch:
Once validated, the load balancer or Kubernetes Service selector is updated to route traffic from Blue to Green.

# Monitoring and Rollback:
Metrics, alerts, and logs are monitored for anomalies. If an issue occurs, traffic can be switched back instantly to the Blue environment.

# Cleanup:
After stability is confirmed, the Blue environment is updated or scaled down for the next release cycle.

## Benefits

- Instant rollback in case of failures.

- Zero downtime during deployment.

- Parallel environments for testing and verification.

- Simplified disaster recovery and reduced risk of partial updates.



## CI/CD Automation
## Continuous Integration (CI)
The CI pipeline is implemented using GitHub actions
The pipeline performs the following stages:
- Build: Containerizes the microsrvices using Docker
- Test: Runs unit and integrayion tests to ensure application correctness.
- Code Quality Analysis: Scans code using SonarQube to enfore quality gates.
- Image Push: Pushed the successfully built images to AWS ECR.

## Continuous Delivery (CD)
## Option 1 – GitOps (ArgoCD Integration):
- After Pushing the image, the CI pipeline automatically updates the values.yaml file in the Helm Chart Repository with the new image tag.
- ArgoCD Continuously monitors this repository and detects the update
- ArgoCD synchronizes the change into the Kubernetes cluster, deploying the new image to the environment.


## Option 2 – Direct Helm Deployment:

- The CI pipeline connects to the EKS cluster and runs helm upgrade --install with the new image tag.
- The Helm chart manages the Blue-Green environment internally, switching traffic post-validation.


## Security & Access Control

- AWS OIDC is used for GitHub Actions authentication to AWS without static credentials.
- IAM roles and least-privilege policies control access to ECR and EKS.
- Secrets such as database credentials and API keys are stored securely in AWS Secrets Manager.



## Monitoring and Observability
## Metrics & Monitoring (Prometheus + Grafana)

- Prometheus continuously scrapes application metrics, cluster performance data, and RabbitMQ statistics using exporters.

- Grafana provides customizable dashboards for visualizing system health, response times, throughput, queue depth, and resource usage.

- Kube-State-Metrics and Node Exporter supply cluster and node data.

- RabbitMQ Exporter tracks message rates, consumer counts, and queue sizes.

- The system defines threshold-based alert rules for service downtime, high latency, and queue overloads from the Helm Charts.


## Alerting and Incident Management

- Alertmanager processes alerts generated by Prometheus.

- Critical alerts (e.g., RabbitMQ queue overload, microservice downtime, high CPU usage) are routed to a Slack channel via webhook integration.

- Alerts are grouped and throttled to prevent noise and include detailed annotations to assist incident response.
