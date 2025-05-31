# Security Documentation for Docker Security Falcon Project

This document outlines the security practices, measures, and issues addressed during the development of the **Docker Security Falcon** project. It is designed to demonstrate a comprehensive understanding of container security principles, CI integration, and CVE mitigation.

---

## Minimal Base Image and Explicit Version Pinning

The base image was explicitly pinned to `python:3.11.12-slim` to ensure deterministic builds, reproducibility, and reduction of vulnerabilities introduced by implicit upgrades.

```dockerfile
FROM python:3.11.12-slim
```

**Why it matters:**

* Avoids using `latest` or floating tags.
* Ensures reproducible builds and secure dependency versions.
* Prevents sudden CVEs from upstream base images.

---

## Non-Root User Implementation

To reduce the container’s attack surface, a non-root user was created with explicit permissions:

```dockerfile
RUN groupadd -r app && useradd -r -g app app \
    && mkdir /app && chown -R app:app /app
USER app
```

**Outcome:** This avoids privilege escalation inside the container, a critical security concern in production environments.

---

## Dependency Pinning and Validation

All dependencies were explicitly version-pinned in `requirements.txt` using:

```bash
pip freeze | grep -E 'fastapi|uvicorn|pydantic|anyio' > requirements.txt
```

Example snippet:

```text
fastapi==0.115.12
uvicorn==0.34.2
pydantic==2.11.5
```

This ensured consistency and removed transient dependency vulnerabilities.

---

## CI-Based CVE Scanning with Trivy

The GitHub Actions workflow `container-security.yaml` uses Aqua Security’s Trivy for continuous CVE scanning:

```yaml
- name: Run Trivy CVE scanner
  uses: aquasecurity/trivy-action@0.14.0
  with:
    image-ref: ${{ secrets.DOCKERHUB_USERNAME }}/falcon-api:${{ github.sha }}
    format: table
    output: reports/trivy-report.txt
    severity: CRITICAL,HIGH
```

**Example Result:**
Prior to pinning `python:3.11-slim`, the scan detected 177 CVEs. After switching to `python:3.11.12-slim`, only 6 CVEs remained, 4 of which were system-level and 2 related to `setuptools`, which are being tracked.

---

## Dockerfile Linting with Hadolint

CI included linting to enforce Dockerfile best practices:

```yaml
- name: Lint Dockerfile with Hadolint
  uses: hadolint/hadolint-action@v3.1.0
  with:
    dockerfile: ./Dockerfile
```

**Detected Warnings Resolved:**

* Added `--no-cache-dir` to `pip install`.
* Avoided use of `latest`.
* Used explicit `USER` declaration.

---

## Principle of Least Privilege in GitHub Workflow

```yaml
permissions:
  contents: read
  security-events: write
  actions: read
  packages: read
```

Each permission was carefully scoped. For instance:

* `security-events: write` allows Trivy to upload results.
* `contents: read` ensures the workflow only reads code, not writes.

---

## Error Handling and Debugging

During development, several critical issues were encountered:

### ❌ Error: `No such option: --host 0.0.0.0`

**Root Cause:** Incorrect `CMD` array syntax.
**Fix:** Ensured proper JSON array format in `CMD`.

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## Build Hygiene

To prevent bloating and reduce attack vectors, the following practices were applied:

* Cleaned up layer cache (`apt-get clean` where applicable)
* Avoided writing to root directories
* Used `COPY` in correct order to optimise layer caching

---

## Conclusion

Security was deeply integrated from development through CI. With pinned dependencies, rootless execution, CVE scanning, and Dockerfile linting, this project aligns with modern container security best practices.

For full implementation and workflow, see [GitHub Repository](https://github.com/JThomas404/docker-security-falcon).
