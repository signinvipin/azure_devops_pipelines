# azure-devops-pipelines

Azure DevOps CI/CD for a Spring Boot (Java 17) app: build with Maven, build and push a Docker image, and deploy to Kubernetes. Built first with the Classic editor, then exported to YAML. This is a lab/learning project.

## Pipeline flow
1. Check out the code from GitHub (`main`)
2. Maven: `clean package -DskipTests`
3. Docker build and push to the registry, tagged with `$(Build.BuildId)` and `latest`
4. `kubectl rollout restart` of the `java-app` deployment
5. `kubectl apply` of the Kubernetes manifest

The pipeline runs on a **self-hosted agent** (pool `az-classic-pool`) on a CentOS ARM VM.

## Dev/test vs production
| Manifest | Image tag | Why |
|---|---|---|
| `k8s/app-deployment-dev-test.yaml` | `latest` | Convenient for development and testing |
| `k8s/app-deployment-prod.yaml` | `$(Build.BuildId)` | Versioned tags give traceability; a `sed` step in the pipeline sets the tag |

## Structure
```text
.
├── pipelines/
│   └── azure-pipelines.yml                  # YAML export of the classic pipeline
├── k8s/
│   ├── app-deployment-dev-test.yaml         # Deployment + Service (latest tag)
│   └── app-deployment-prod.yaml             # Deployment + Service (versioned tag)
├── docs/
│   └── azure-classic-pipeline-setup.txt     # Agent VM, service connections, classic editor tasks
├── Dockerfile                               # Alpine + OpenJDK 17 image for the JAR
├── pom.xml
└── src/                                     # Sample Spring Boot app
```

## Setup (summary)
- A VM with Java 17, Maven, Git and Docker, running the Azure Pipelines agent
- Service connections: GitHub, Docker registry, Kubernetes
- Full steps are in `docs/azure-classic-pipeline-setup.txt`

## Notes
- Service connection IDs, the Azure DevOps organization name and private IPs are masked with `xx`. Use your own values.
- The prod manifest's `$(Build.BuildId)` placeholder is replaced by the pipeline, so it is not meant to be applied by hand.
