# sonarqube-local

**Reproducible DevSecOps: Deploying SonarQube Locally with Kubernetes and GitHub Actions**

### 🎯 Project Objective

This artefact showcases a reproducible, modular deployment of **SonarQube**—a leading static code analysis platform—on a local **Kubernetes cluster** powered by **Docker Desktop**. The deployment is orchestrated using declarative Kubernetes manifests and integrated into a **CI/CD pipeline** via **GitHub Actions**, executed on a **self-hosted runner**. 

The primary goal is to demonstrate how DevSecOps principles—such as early defect detection, continuous inspection, and secure delivery—can be embedded into local development workflows. By combining container orchestration with automated code quality checks, this setup enables developers and educators to validate infrastructure, reinforce secure coding practices, and promote reproducibility across environments.

### 🔍 **Why This Matters**

- **Reproducibility**: All components—from Kubernetes manifests to GitHub workflows—are version-controlled and portable, ensuring consistent behavior across machines and teams.
- **Security & Quality**: SonarQube provides real-time feedback on code vulnerabilities, bugs, and maintainability issues, aligning with DevSecOps goals.
- **Local-first CI/CD**: Running GitHub Actions on a self-hosted runner allows full control over execution, ideal for offline development, classroom demos, or constrained environments.
- **Educational Value**: The artefact is designed to be outreach-ready, enabling STEM learners to explore containerization, CI/CD, and secure software engineering in a hands-on, annotated format.


### 🛠️ Tools & Technologies
- **Kubernetes (Docker Desktop)**
- **SonarQube (Community Edition)**
- **GitHub Actions (Self-hosted runner)**
- **YAML manifests (K8s native)**
- **NodePort service for local access**
- **PersistentVolumeClaim for data durability**


### ✅ Validation Checklist
- [x] SonarQube accessible at `http://localhost:30900`
- [x] Persistent data stored via PVC
- [x] GitHub Actions triggered on push
- [x] Static analysis results visible in SonarQube dashboard
- [x] Reproducible across machines with Docker Desktop and K8s enabled

