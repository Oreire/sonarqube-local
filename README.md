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

ingress deployed manually
Codes
name: SonarQube CI/CD

on:
  push:
    branches: [ main ]

jobs:
  deploy-and-scan:
    runs-on: self-hosted

    steps:
      # Step 1: Checkout code
      - name: Checkout code
        uses: actions/checkout@v3

      # Step 2: Setup kubectl
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'latest'

      # Step 3: Configure Kubernetes context (Docker Desktop)
      - name: Configure Kubernetes Context
        shell: pwsh
        run: |
          kubectl config use-context docker-desktop
          kubectl cluster-info

      # Step 4: Deploy SonarQube with Ingress
      - name: Deploy SonarQube
        shell: pwsh
        run: |
          kubectl apply -f Kube/sonar.yaml
          kubectl rollout status deployment/sonarqube -n sonarqube --timeout=300s
    
      - name: Check SonarQube Pod Status
        shell: pwsh
        run: |
            kubectl get pods -n sonarqube
            kubectl get svc -n sonarqube
      
      - name: Start SonarQube Port-Forward
        shell: pwsh
        run: |
          Start-Process -NoNewWindow -FilePath "kubectl" -ArgumentList "port-forward --address 127.0.0.1 service/sonarqube-service -n sonarqube 10090:9000"
          
          # Step 5: Install Sonar Scanner (Windows) with folder cleanup

      - name: Wait for SonarQube to be reachable
        shell: pwsh
        run: |
            $maxAttempts = 10
            for ($i = 0; $i -lt $maxAttempts; $i++) {
             try {
             $response = Invoke-WebRequest -Uri "http://localhost:10090/api/system/health" -UseBasicParsing
            if ($response.StatusCode -eq 200) {
             Write-Host "SonarQube is ready"
             break
            }
            } catch {
            Write-Host "Waiting for SonarQube..."
            Start-Sleep -Seconds 5
            }
            }

      - name: Install Sonar Scanner
        shell: pwsh
        run: |
          $url = "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-windows.zip"
          $zipPath = "$env:USERPROFILE\sonar-scanner.zip"
          $extractPath = "$env:USERPROFILE\sonar-scanner"

          # Remove previous installation if it exists
          if (Test-Path $extractPath) {
              Remove-Item -Recurse -Force $extractPath
          }

          Invoke-WebRequest -Uri $url -OutFile $zipPath
          Expand-Archive -LiteralPath $zipPath -DestinationPath $extractPath -Force

          # Add to PATH for this workflow
          echo "$extractPath\sonar-scanner-5.0.1.3006-windows\bin" >> $env:GITHUB_PATH

      # Step 6: Run SonarQube Scan with Token Pre-flight Check
    
      - name: Run SonarQube Scan
        shell: pwsh
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: |
          if (-not $env:SONAR_TOKEN) {
            throw "SONAR_TOKEN is not set"
          }

            sonar-scanner `
            -D"sonar.projectKey=sonarqube-local" `
            -D"sonar.sources=." `
            -D"sonar.host.url=http://localhost:10090" `
            -D"sonar.login=$env:SONAR_TOKEN"

            