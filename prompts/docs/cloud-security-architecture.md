# Cloud Security Architecture (Mermaid Example)

This file demonstrates a **cloud security architecture** using Mermaid diagrams.
It can be used as a reference or template for your GitHub repo documentation.

---

## Mermaid Diagram Example

```mermaid
flowchart TD
    %% ===============================
    %% Developer & CI/CD
    %% ===============================

    DEV[Developer<br/>(Code / Commits)]
    CICD[CI/CD Pipeline<br/>- Linting<br/>- Unit Tests<br/>- IaC Scans]

    DEV --> CICD

    %% ===============================
    %% Security Scanning & Supply Chain
    %% ===============================

    SCAN[Vulnerability Scanning<br/>(Trivy / Grype)]
    SBOM[SBOM Generation<br/>(Syft)]
    SIGN[Image Signing<br/>(cosign)]

    CICD --> SCAN
    CICD --> SBOM
    CICD --> SIGN

    %% ===============================
    %% Hardened Image Artifact
    %% ===============================

    IMAGE[Hardened Container Image]

    SCAN --> IMAGE
    SBOM --> IMAGE
    SIGN --> IMAGE

    %% ===============================
    %% ECR
    %% ===============================

    ECR[AWS ECR<br/>- Image Scanning<br/>- Lifecycle Policies<br/>- Signature Verification]

    IMAGE -->|Push Signed Image| ECR

    %% ===============================
    %% EKS & ECS Branches
    %% ===============================

    EKS[AWS EKS Cluster<br/>- IRSA<br/>- RBAC<br/>- NetworkPolicies<br/>- PSS Restricted]
    ECS[AWS ECS Fargate<br/>- Least-Priv Task Role<br/>- Private Subnets<br/>- Secrets Manager]

    ECR --> EKS
    ECR --> ECS

    %% ===============================
    %% Application Workloads
    %% ===============================

    APP_EKS[EKS Application<br/>(Pods/Deployments)]
    APP_ECS[ECS Application<br/>(Tasks/Services)]

    EKS --> APP_EKS
    ECS --> APP_ECS

    %% ===============================
    %% Logging & Monitoring
    %% ===============================

    LOGS[CloudWatch / K8s Logs]
    MON[Monitoring<br/>(Metrics / ALB / SGs)]

    APP_EKS --> LOGS
    APP_ECS --> LOGS

    APP_EKS --> MON
    APP_ECS --> MON

    %% ===============================
    %% End User
    %% ===============================

    USER[End User<br/>via ALB]

    MON --> USER
```
