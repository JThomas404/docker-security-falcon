# Docker Container Security Falcon

## Table of Contents

- [Overview](#overview)
- [Real-World Business Value](#real-world-business-value)
- [Prerequisites](#prerequisites)
- [Project Folder Structure](#project-folder-structure)
- [Tasks and Implementation Steps](#tasks-and-implementation-steps)
- [Core Implementation Breakdown](#core-implementation-breakdown)
- [Local Testing and Debugging](#local-testing-and-debugging)
- [Design Decisions and Highlights](#design-decisions-and-highlights)
- [Errors Encountered and Resolved](#errors-encountered-and-resolved)
- [Skills Demonstrated](#skills-demonstrated)
- [Conclusion](#conclusion)
- [Optional Enhancements](#optional-enhancements)

---

## Overview

This project implements a security-first approach to containerising a [**FastAPI application**](https://github.com/JThomas404/fastapi-project), integrating automated vulnerability scanning, Dockerfile linting, and secure CI/CD practices. The implementation demonstrates practical application of DevSecOps principles through automated security controls, least-privilege container design, and comprehensive vulnerability management.

The solution transforms a standard FastAPI application into a production-ready, security-hardened container image with automated scanning and publishing workflows. The architecture emphasises security at every layer, from base image selection to runtime user permissions.

---

## Real-World Business Value

This implementation addresses critical enterprise security requirements:

- **Risk Mitigation**: Automated CVE detection prevents deployment of vulnerable container images to production environments
- **Compliance Alignment**: Implements security controls required for SOC 2, ISO 27001, and similar compliance frameworks
- **Operational Efficiency**: Reduces manual security review overhead through automated scanning and reporting
- **Supply Chain Security**: Ensures container images meet security baselines before publication to registries
- **Cost Reduction**: Prevents security incidents through proactive vulnerability management and secure-by-design practices

The automated pipeline reduces security review cycles from days to minutes while maintaining comprehensive security coverage.

---

## Prerequisites

- Docker Engine 20.10+ with BuildKit support
- GitHub account with Actions enabled
- Docker Hub account for image registry
- Local development environment with Python 3.11+

---

## Project Folder Structure

```
docker-security-falcon/
├── app/
│   ├── main.py                     # FastAPI application entry point
│   └── models.py                   # Pydantic data models
├── .github/
│   └── workflows/
│       └── container-security.yaml # CI/CD pipeline with security scanning
├── reports/
│   ├── trivy-report.txt            # Latest vulnerability scan results
│   └── trivy-scan.txt              # Historical scan data
├── Dockerfile                      # Secure container build
├── requirements.txt                # Pinned Python dependencies
├── .dockerignore                   # Build context optimisation
├── SECURITY.md                     # Security documentation
└── README.md                       # Project documentation
```

---

## Tasks and Implementation Steps

1. **Security-First Dockerfile Design**

   - Implemented non-root user execution with dedicated system user
   - Applied minimal base image strategy using Python slim variant
   - Enforced dependency pinning for reproducible builds

2. **Automated Vulnerability Management**

   - Integrated Trivy scanner for comprehensive CVE detection
   - Configured severity filtering for CRITICAL and HIGH vulnerabilities
   - Implemented automated report generation and artifact storage

3. **CI/CD Pipeline Development**

   - Built GitHub Actions workflow with security-first approach
   - Integrated Hadolint for Dockerfile best practice validation
   - Automated Docker Hub publishing with SHA-based tagging

4. **Security Controls Implementation**
   - Applied least-privilege principles throughout container lifecycle
   - Implemented secure secrets management for registry authentication
   - Configured proper file permissions and ownership

---

## Core Implementation Breakdown

### Secure Dockerfile Architecture

The [Dockerfile](https://github.com/JThomas404/docker-security-falcon/blob/main/Dockerfile) implements multiple security layers:

```dockerfile
FROM python:3.11.12-slim

WORKDIR /app

# Create non-root user with minimal privileges
RUN addgroup --system pygroup && adduser --system --ingroup pygroup pyuser

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY ./app /app/app

# Apply secure file ownership
RUN chown -R pyuser:pygroup /app

# Switch to non-root user
USER pyuser

EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Security Features**:

- Explicit base image pinning (`python:3.11.12-slim`) prevents supply chain attacks
- Non-root user execution reduces container escape attack surface
- Minimal package installation reduces vulnerability exposure
- Proper file ownership ensures runtime security

### CI/CD Security Pipeline

The [GitHub Actions workflow](https://github.com/JThomas404/docker-security-falcon/blob/main/.github/workflows/container-security.yaml) implements comprehensive security automation:

```yaml
- name: Run Trivy CVE Scanner
  uses: aquasecurity/trivy-action@0.14.0
  with:
    image-ref: ${{ secrets.DOCKERHUB_USERNAME }}/falcon-api:${{ github.sha }}
    format: table
    output: reports/trivy-report.txt
    severity: CRITICAL,HIGH

- name: Lint Dockerfile with Hadolint
  uses: hadolint/hadolint-action@v3.1.0
  with:
    dockerfile: Dockerfile
```

**Pipeline Security Controls**:

- Pre-deployment vulnerability scanning prevents vulnerable image promotion
- Dockerfile linting enforces security best practices
- Automated report generation provides audit trail
- SHA-based tagging ensures image immutability

### FastAPI Application Structure

The application implements clean architecture principles with [main.py](https://github.com/JThomas404/docker-security-falcon/blob/main/app/main.py) handling routing and [models.py](https://github.com/JThomas404/docker-security-falcon/blob/main/app/models.py) defining data structures using Pydantic for type safety and validation.

---

## Local Testing and Debugging

### Container Build and Execution

```bash
# Build the secure container image
docker build -t falcon-api .

# Run with security context
docker run -d -p 8000:8000 --name falcon-container falcon-api

# Verify non-root execution
docker exec falcon-container whoami
# Output: pyuser

# Access API documentation
curl http://localhost:8000/docs
```

### Security Validation

```bash
# Run local Trivy scan
trivy image falcon-api

# Validate Dockerfile security
hadolint Dockerfile

# Test API endpoints
curl -X POST "http://localhost:8000/todos" \
  -H "Content-Type: application/json" \
  -d '{"id": 1, "item": "Test security implementation"}'
```

### Container Security Verification

```bash
# Inspect container security context
docker inspect falcon-container | grep -A 10 "SecurityOpt"

# Verify file permissions
docker exec falcon-container ls -la /app
```

---

## Design Decisions and Highlights

### Base Image Selection

Selected `python:3.11.12-slim` over Alpine or full Python images based on:

- **Security**: Regular Debian security updates and established CVE management
- **Compatibility**: Broad library support without compilation requirements
- **Size**: Minimal footprint while maintaining functionality
- **Maintenance**: Long-term support and predictable update cycles

### Non-Root User Implementation

Implemented dedicated system user rather than using existing users:

- **Isolation**: Prevents privilege escalation through shared user contexts
- **Auditability**: Clear ownership and permission boundaries
- **Compliance**: Meets enterprise security requirements for container execution

### Vulnerability Scanning Strategy

Configured Trivy to focus on CRITICAL and HIGH severity vulnerabilities:

- **Risk-Based**: Prioritises vulnerabilities with actual exploitation potential
- **Operational**: Reduces noise while maintaining security coverage
- **Automated**: Integrates seamlessly into CI/CD workflow without manual intervention

### Dependency Management

Pinned all dependencies to specific versions in [requirements.txt](https://github.com/JThomas404/docker-security-falcon/blob/main/requirements.txt):

- **Reproducibility**: Ensures consistent builds across environments
- **Security**: Prevents automatic updates that could introduce vulnerabilities
- **Compliance**: Provides audit trail for dependency management

---

## Errors Encountered and Resolved

### Docker Hub Authentication Issues

**Problem**: Initial pipeline failures due to Docker Hub authentication errors.

**Solution**: Implemented proper secrets management using GitHub repository secrets for `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`, ensuring secure credential handling without exposure in logs.

### Trivy Report Generation

**Problem**: Trivy reports were not being generated consistently in CI environment.

**Solution**: Added explicit directory creation step (`mkdir -p reports`) before Trivy execution and configured proper artifact upload with retention policies.

### File Permission Conflicts

**Problem**: Container startup failures due to permission conflicts between build and runtime users.

**Solution**: Implemented explicit ownership transfer using `chown -R pyuser:pygroup /app` after file copying but before user context switch.

---

## Skills Demonstrated

### Container Security

- Secure Dockerfile design with multi-layer security controls
- Non-root user implementation and privilege management
- Vulnerability scanning integration and automated reporting
- Supply chain security through dependency pinning

### DevSecOps Implementation

- Security-first CI/CD pipeline design
- Automated security testing and validation
- Compliance-ready documentation and audit trails
- Risk-based vulnerability management

### Infrastructure as Code

- GitHub Actions workflow development
- Automated container registry management
- Secret management and secure authentication
- Pipeline optimisation and error handling

### Software Engineering

- FastAPI application development with type safety
- Clean architecture implementation
- API design and documentation
- Testing and validation strategies

---

## Conclusion

This project demonstrates practical implementation of enterprise-grade container security practices, combining automated vulnerability management with secure development workflows. The solution provides a foundation for production container deployments while maintaining security, compliance, and operational efficiency.

The implementation showcases ability to balance security requirements with development velocity, creating automated systems that enhance rather than impede the development process. The comprehensive documentation and testing approach ensures maintainability and knowledge transfer.

---

## Optional Enhancements

- **Multi-stage Build Optimisation**: Implement builder pattern to reduce final image size
- **Runtime Security Monitoring**: Integrate Falco or similar runtime security tools
- **SBOM Generation**: Add Software Bill of Materials generation for supply chain transparency
- **Security Policy as Code**: Implement OPA/Gatekeeper policies for Kubernetes deployment
- **Advanced Scanning**: Integrate additional scanners (Snyk, Anchore) for comprehensive coverage
- **Secrets Scanning**: Add GitLeaks or TruffleHog for repository secret detection

---
