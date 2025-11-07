flowchart LR
 subgraph DEV["Developer / CI Repos"]
        code["Service Code & Helm Charts"]
        config["Config Repo values.yaml"]
  end
 subgraph CDOptions["CD Options"]
        CI_Helm["CI -> Helm CLI optional"]
        ArgoCD["ArgoCD GitOps -> Cluster"]
  end
 subgraph Apps["Microservices"]
        Order["Order Service"]
        Inventory["Inventory Service"]
        Payment["Payment Service"]
        Notif["Notification Service"]
  end
 subgraph AWS["EKS Cluster prod namespace foodelo"]
    direction TB
        ALB["ALB / Ingress Controller ALB / NGINX"]
        Ingress["Ingress"]
        Apps
        Rabbit[("RabbitMQ StatefulSet + PVs")]
        DB[("Postgres / Managed RDS")]
        Prom["Prometheus + exporters"]
        Graf["Grafana"]
        AM["Alertmanager -> Slack / PagerDuty"]
        ArgocdApp["ArgoCD in-cluster"]
  end
    code -- push --> CI["GitHub Actions CI"]
    CI -- build & push --> ECR[("ECR / Container Registry")]
    CI -- "update values.yaml" --> config
    config -- git --> ArgoRepo["ArgoCD watches this repo"]
    ECR -- image --> CI_Helm
    ArgoRepo --> ArgoCD
    CI_Helm -- "helm upgrade --install" --> EKS["EKS"]
    ArgoCD -- "auto-sync" --> EKS
    Ingress --> ALB
    ALB --> Apps
    Apps -- AMQP --> Rabbit
    Apps -- SQL --> DB
    Apps -- metrics --> Prom
    Prom --> Graf & AM
    ArgocdApp --> Apps
    AWS --> n1["Kuberntes Cluster"]

    classDef infra fill:#f3f4f6,stroke:#666,stroke-width:1px



