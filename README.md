3-Tier Voting App on Amazon EKS
Containerized microservices deployed on AWS EKS with GitHub Actions CI/CD

📌 Project Overview
A distributed voting application demonstrating DevOps practices:

. Frontend (Vote): Python/Flask (vote/)

. Queue: Redis

. Worker: .NET 7.0 (worker/)

. Database: PostgreSQL (secured via config updates)

. Results (Result): Node.js/Express (result/)

Key Achievements:
✅ Deployed on AWS EKS using eksctl (see cluster config).
✅ Automated CI/CD with GitHub Actions (18 workflow runs).
✅ Hardened PostgreSQL security (updated deployment configs).

🛠️ Tech Stack
Category	Tools
Cloud	         -  AWS EKS, EC2, IAM, VPC
Orchestration  - 	Kubernetes (kubectl), Docker Compose
CI/CD	         -  GitHub Actions (Build/Deploy)
Languages     -	Python (Flask), .NET 7.0, Node.js, JavaScript
Database	     -   PostgreSQL (with security optimizations)

🚀 Deployment
1. Local Development

docker compose up  # Runs all services (vote, redis, worker, db, result)
Access:
Vote: http://localhost:8080
Results: http://localhost:8081

2. AWS EKS Cluster Setup
eksctl create cluster --name <cluster> --node-type t3.medium --nodes 3 --region <region>
kubectl apply -f k8s/  # Applies Kubernetes manifests

Note for ARM64 (M1/M2):
docker buildx build --platform linux/amd64 -t myorg/worker:latest ./worker

⚙️ CI/CD Pipeline
GitHub Actions Workflow:

Triggers: On push to main

Steps:

Build Docker images for vote, worker, result.

Push to container registry (e.g., ECR).

Deploy to EKS via kubectl.


🔒 Security Highlights
PostgreSQL: Updated security configs in postgres-deployment.yaml.

IAM: Least-privilege roles for EKS nodes.

Secrets: (Optional) Recommend adding AWS Secrets Manager for credentials.

📂 Repository Structure
plaintext
├── .github/workflows/  # GitHub Actions CI/CD (e.g., deploy.yml)
├── vote/               # Python/Flask voting interface
├── result/             # Node.js results dashboard
├── worker/             # .NET 7.0 vote processor
├── k8s/                # Kubernetes manifests (if applicable)
├── docker-compose.yml  # Local dev environment
└── eksctl-config/      # EKS cluster config (if used)


🔥 Key Challenges & Technical Solutions
 
1. Database Security Configuration
Challenge: Default PostgreSQL deployment had unencrypted traffic and excessive permissions.
Solution:
> Implemented Kubernetes Secrets for credential management
> Added securityContext restrictions in deployment:

yaml:
securityContext:
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
> Configured network policies to limit DB access to only worker pods

2. CI/CD Pipeline Reliability
Challenge: Silent deployment failures caused service interruptions.
Solution:
> Implemented GitHub Actions health checks:

yaml:
- name: Verify deployment
  run: |
    kubectl rollout status deployment/vote-app --timeout=2m
    curl -sSf http://$LOAD_BALANCER_URL/health
  
> Added automatic rollback on failure:
kubectl rollout undo deployment/vote-app

3. Multi-Architecture Support
Challenge: ARM-built containers failed on AWS x86 nodes.
Solution:
> Configured cross-platform builds: docker buildx build --platform linux/amd64 -t app/worker:latest ./worker
  
> Added architecture validation step in CI pipeline

4. Redis Performance Bottlenecks
Challenge: High traffic caused vote processing delays.
Solution:
> Implemented Horizontal Pod Autoscaler for workers:

yaml:
metrics:
  - type: External
    external:
      metricName: redis_queue_length
      targetAverageValue: 100
> Optimized .NET worker batch processing intervals

5. Configuration Management
Challenge: Environment-specific settings required manual updates.
Solution:
> Implemented Kustomize overlays for dev/prod environments

Automated config injection using:
kubectl create configmap app-config --from-env-file=.env
