# Java Application DevOps Pipeline

A hands-on DevOps project demonstrating a Jenkins-based CI/CD workflow for building, analyzing, packaging, containerizing, scanning, and deploying a Java web application.

**Repository:** https://github.com/Chakri-79/Java-app-devops-pipeline

## Technology Stack

- **Source control:** Git, GitHub
- **CI/CD:** Jenkins Pipeline
- **Build and testing:** Maven
- **Code quality:** SonarQube and Quality Gate
- **Artifact management:** Nexus Repository
- **Containerization:** Docker
- **Image vulnerability scanning:** Trivy
- **Container registry:** Docker Hub
- **Deployment:** Docker Swarm using Docker Stack
- **Database:** MySQL
- **Infrastructure:** AWS EC2

## Infrastructure

| Server               | Instance type    | Purpose                                  | Tools / responsibilities                          |
|----------------------|------------------|------------------------------------------|---------------------------------------------------|
| Jenkins server       | `m7i-flex.large` | CI/CD orchestration                      | Jenkins,SonarQube, Java/JDK, Docker,Nexus         |
| Docker Swarm manager | `m7i-flex.large` | Manage the Swarm cluster and deployments | Docker Engine,Java,Git Docker Swarm,Trivy         |
| Swarm worker node(s) | `t3.micro`       | Run services scheduled by the manager    | Docker Engine and application/supporting services |



### Infrastructure flow

```text
GitHub Repository
       |
       v
Jenkins EC2 (m7i-flex.large)
Jenkins | SonarQube | JDK | Docker | Nexus
       |
       v
CI/CD Pipeline
Build -> SonarQube -> Nexus -> Docker Build -> Trivy Scan
       |
       v
Docker Hub
       |
       v
Docker Swarm Manager (m7i-flex.large)
       |
       v
Swarm Worker Node(s)
       |
       v
Java Application + MySQL
```

## Jenkins Plugins and Configuration

Plugin / configuration                               | Purpose                                       |
|----------------------------------------------------|-----------------------------------------------|
**Pipeline Stage View**                              | Displays pipeline stages and their build status 
**Maven tool installation** (Manage Jenkins → Tools) | Makes Maven available to pipeline jobs 
**SonarQube Scanner**                                | Runs code analysis 
**SonarQuality Gates plugin**                        | Connects Jenkins with the SonarQube server and quality-gate workflow 
**Nexus Artifact Uploader**                          | Uploads the packaged WAR to Nexus 
**pipeline utility Steps**                           |Pipeline script reading `pom.xml` | Retrieves artifact details such as `groupId`, `artifactId`, and `version` 
**Docker registry endpoint**                         | Authenticates with the configured image registry 


*Maven installation and registry endpoint configuration are Jenkins configuration items, not standalone plugins. Plugin display names may vary by installed version.*

## CI/CD Workflow

```text
GitHub
  |
  v
1. CODE: Checkout source
  |
  v
2. BUILD & TEST: mvn clean package
  |
  v
3. CODE QUALITY: SonarQube analysis
  |
  v
4. QUALITY GATE: Verify the quality-gate result
  |
  v
5. ARTIFACT PUBLISH: Upload the WAR to Nexus
  |
  v
6. DOCKER BUILD: Build the application image
  |
  v
7. SECURITY SCAN: Scan the image with Trivy
  |
  v
8. REGISTRY: Push the image to Docker Hub
  |
  v
9. DEPLOY: Deploy services with Docker Stack
  |
  v
10. VALIDATE: Check services and application endpoint
```

### Pipeline stages

1. **Code checkout:** Jenkins retrieves source code from GitHub.
2. **Build and test:** Maven compiles the Java application, runs configured tests, and packages the WAR file.
3. **Static code analysis:** SonarQube analyzes the code and reports findings.
4. **Quality Gate:** Jenkins checks the SonarQube Quality Gate before proceeding.
5. **Artifact publishing:** The WAR is published to Nexus. The pipeline script retrieves metadata such as `groupId`, `artifactId`, and `version` from `pom.xml`.
6. **Docker image build:** Docker builds the application image using the project’s Docker configuration.
7. **Image security scan:** Trivy scans the image for known vulnerabilities.
8. **Image publishing:** The image is pushed to the configured Docker Hub repository.
9. **Deployment:** Docker Stack deploys the application and supporting services to Docker Swarm.
10. **Post-deployment validation:** Confirm that services are running and test the application endpoint.

> Only mark a stage as successful after confirming its Jenkins result or validating the deployment.

### Post-Build Email Notifications

Jenkins is configured to send an email notification to the configured recipient addresses after the pipeline completes. The notification communicates the pipeline/build status so the relevant people can track successful and failed runs without manually checking Jenkins.

**Trigger:** After the pipeline build completes (post-build action).

**Recipients:** The email addresses configured in the Jenkins job.

**Purpose:** Share the pipeline result and help the team follow up on failures or successful deployments.

Email delivery depends on the Jenkins email/SMTP configuration and the recipient addresses configured for the job.

## Repository Structure

```text
Java-app-devops-pipeline/
├── src/                  # Java source and tests
├── pom.xml               # Maven build configuration
├── Docker-app/           # Application image build files
├── Docker-db/            # Database image/configuration files
├── manifests/            # Deployment manifests, if used
├── compose.yml           # Container/service configuration
├── compose2.yml          # Additional Compose configuration, if used
├── Jenkinsfile           # Jenkins pipeline as code
├── screenshots/          # Pipeline and deployment evidence
└── README.md             # Project documentation
```


## Prerequisites

- Compatible JDK and Maven installations
- Jenkins with Pipeline support and the required plugins
- SonarQube server and scanner integration
- Nexus Repository with a repository configured for WAR artifacts
- Docker Engine and Docker Swarm access
- Trivy
- Docker Hub repository
- MySQL and other application services required by the project

## Jenkins Configuration

Configure tools and credentials in Jenkins rather than hard-coding secrets in the repository. Typical configuration includes:

- JDK and Maven installations
- SonarQube server and token credential
- Nexus URL and publishing credentials
- Docker Hub username and access-token credential
- Docker registry endpoint
- Docker/Swarm deployment access

Use Jenkins credential IDs in the `Jenkinsfile`. Never commit passwords, access tokens, private keys, or secret-containing environment files.

## Build Locally

From the repository root:

```bash
mvn clean package
```

Check the generated WAR filename under `target/` before using it in Docker or Nexus stages. Review the Dockerfiles and Compose files before deploying to ensure service names, ports, image names, and network settings match your environment.

## Validation Checklist

- [ ] Jenkins checks out the intended branch and commit.
- [ ] `mvn clean package` completes and tests pass.
- [ ] SonarQube analysis completes and the Quality Gate passes.
- [ ] The WAR is uploaded to the expected Nexus repository.
- [ ] The Docker image builds successfully.
- [ ] Trivy scanning completes and findings are reviewed.
- [ ] The image is pushed to the intended Docker Hub repository.
- [ ] Docker Swarm services reach the expected replica count.
- [ ] The application endpoint and key flows are tested.
- [ ] MySQL and other required services are reachable by the application.

## Troubleshooting

- **Nexus publishing fails:** Check the repository URL, artifact filename/path, Jenkins credential ID, and publishing permissions.
- **A supporting service hostname cannot be resolved:** Verify that the hostname matches the actual service name and that the containers share the intended Docker network.
- **The application returns HTTP 500 during a database operation:** Check database health, network connectivity, credentials, schema initialization, and JDBC configuration.
- **Deployment validation fails:** Inspect Docker service status, logs, replica counts, port mappings, and endpoint configuration.

## Security and Maintenance

- Store credentials in Jenkins Credentials or an approved secret manager.
- Prefer specific image tags for reproducible deployments.
- Review Trivy findings before publishing or deploying an image.
- Avoid committing generated artifacts, secrets, and unnecessary build output.
- Redact sensitive details in screenshots.
- Update this README whenever the pipeline, infrastructure, plugins, or deployment configuration changes.

## Author

**Chakradhar**

GitHub: https://github.com/Chakri-79
