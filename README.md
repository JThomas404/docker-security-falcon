# Docker Container Security Falcon (Phase 1)

This project was designed to take a [**FastAPI application**](https://github.com/JThomas404/fastapi-project) and containerise the API with Docker best practices, scanned for vulnerabilities using Trivy, and integrated with GitHub Actions for automated CVE analysis, Dockerfile linting, and lastly publishing the image to Docker Hub.

---

## Table of Contents

* [Objectives](#objectives)
* [Technology Rationale](#technology-rationale)
* [DevSecOps Lifecycle](#devsecops-lifecycle)
* [Key Security Features](#key-security-features)
* [Technologies Used](#technologies-used)
* [Folder Structure](#folder-structure)
* [Build & Run the App Locally](#build--run-the-app-locally)
* [GitHub Actions CI/CD Pipeline](#github-actions-cicd-pipeline)

  * [CI Workflow Summary](#ci-workflow-summary)
  * [Workflow Snippet – Trivy CVE Scanner](#workflow-snippet--trivy-cve-scanner-githubworkflowscontainer-securityyaml)
* [Trivy CVE Summary](#trivy-cve-summary-as-of-last-scan)
* [Docker Image](#docker-image)
* [Security Notes](#security-notes)

---

## Objectives

* Enforce Docker security best practices—use of a non-root user, explicit image pinning, dependency locking.
* Integrate CVE detection via Trivy to scan during CI before publishing.
* Create a CI pipeline using GitHub Actions to automate linting, scanning, tagging, and pushing.
* Dockerise a working FastAPI application using secure patterns.
* Acquire hands-on proficiency in the secure build lifecycle and container publishing.

---

## Technology Rationale

* **FastAPI** – Efficient and modern web framework supporting async I/O.
* **Docker** – Ensures consistent container environments across systems.
* **Trivy** – Lightweight and fast vulnerability scanner for images and packages.
* **Hadolint** – Validates Dockerfile against community and security standards.
* **GitHub Actions** – Provides a streamlined CI/CD integration for this project.

---

## DevSecOps Lifecycle

This project aligns with each stage of secure container development:

1. **Development** – Structured, modular FastAPI application.
2. **Containerisation** – Dockerfile security practices (non-root user, minimal base image, pinned dependencies).
3. **Security Scanning** – Integrated CVE analysis and linting.
4. **CI/CD Automation** – Fully scripted GitHub Actions workflow.
5. **Publishing** – Hardened Docker images pushed automatically to Docker Hub.

---

## Key Security Features

* **Explicit base image**: `python:3.11.12-slim` for reproducible and minimal builds.
* **Non-root user**: Reduces attack surface and limits container privileges.
* **Dependency pinning**: Python and OS packages are locked to fixed versions.
* **Trivy scanning**: Catches known CVEs before image promotion.
* **Hadolint**: Enforces Dockerfile linting for best practices.
* **CI automation**: GitHub Actions enforces checks on every push and PR.

---

## Technologies Used

* **FastAPI** – Web API development
* **Docker** – Containerisation
* **Trivy** – Vulnerability scanning
* **Hadolint** – Dockerfile linting
* **GitHub Actions** – Continuous integration pipeline

---

## Folder Structure

```
docker-security-falcon/
├── app/
│   ├── main.py
│   └── models.py
├── .github/
│   └── workflows/
│       └── container-security.yaml
├── reports/
│   ├── trivy-report.txt      
│   └── trivy-scan.txt        
├── Dockerfile
├── README.md
├── SECURITY.md
└── requirements.txt
```

---

## Build & Run the App Locally

```bash
# Build the image
docker build -t falcon-api .

# Run the container
docker run -d -p 8000:8000 falcon-api

# Access the FastAPI docs
http://localhost:8000/docs
```

---

## GitHub Actions CI/CD Pipeline

**Trigger**: Executes on push to `main` or on any pull request.

---

### CI Workflow Summary

* Clone source code
* Initialise Docker Buildx
* Authenticate with Docker Hub
* Build and tag the container
* Run Trivy scan and output CVE report
* Upload the report artifact
* Lint Dockerfile via Hadolint

---

### Workflow Snippet – Trivy CVE Scanner (`.github/workflows/container-security.yaml`)

```yaml
- name: Run Trivy CVE scanner
  uses: aquasecurity/trivy-action@0.14.0
  with:
    image-ref: ${{ secrets.DOCKERHUB_USERNAME }}/falcon-api:${{ github.sha }}
    format: table
    output: reports/trivy-report.txt
    severity: CRITICAL,HIGH
```

---

## Trivy CVE Summary (as of last scan)

```
Total Vulnerabilities Found: 6
  - CRITICAL: 2
  - HIGH: 4

Scan completed successfully. Full report: /reports/trivy-report.txt
```

---

## Docker Image

* **Docker Hub**: [zermann/falcon-api](https://hub.docker.com/r/zermann/falcon-api/tags)
* **Tags**: `latest`, `${{ github.sha }}`

---

## Security Notes

Further notes on CVE results, debugging steps, and implementation reasoning are provided in the companion [Security Documentation for Docker Security Falcon Project](./SECURITY.md) document.

---
