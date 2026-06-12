# OWASP ZAP DAST Security Scan with GitHub Actions

## Project Overview

This project demonstrates how to integrate OWASP ZAP Dynamic Application Security Testing (DAST) into a GitHub Actions CI/CD pipeline.

The application is deployed on an AWS EC2 instance and automatically scanned whenever code is pushed to the GitHub repository.

---

## Architecture

Developer Push
↓
GitHub Repository
↓
GitHub Actions
↓
OWASP ZAP Baseline Scan
↓
HTML Report Generation
↓
GitHub Artifacts
↓
Developer Reviews Findings

---

## Prerequisites

### AWS EC2

* Amazon Linux / Ubuntu EC2 Instance
* Security Group allowing:

| Port | Purpose            |
| ---- | ------------------ |
| 22   | SSH                |
| 3000 | Application Access |

Example:

```bash
22/tcp     SSH
3000/tcp   Application
```

---

## Connect to EC2

```bash
ssh -i mykey.pem ec2-user@<EC2-PUBLIC-IP>
```

Example:

```bash
ssh -i mykey.pem ec2-user@54.92.155.35
```

---

## Install Docker

Amazon Linux:

```bash
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
newgrp docker
```

Verify:

```bash
docker --version
```

---

## Run Juice Shop Application

Pull image:

```bash
docker pull bkimminich/juice-shop
```

Run container:

```bash
docker run -d \
--name juice-shop \
-p 3000:3000 \
bkimminich/juice-shop
```

Verify:

```bash
docker ps
```

Access:

```text
http://<EC2-PUBLIC-IP>:3000
```

Example:

```text
http://54.92.155.35:3000
```

---

## Create GitHub Repository

```bash
git init
git remote add origin https://github.com/<username>/owasp-zap-devsecops-lab.git
```

Clone repository:

```bash
git clone https://github.com/<username>/owasp-zap-devsecops-lab.git
cd owasp-zap-devsecops-lab
```

---

## Create GitHub Workflow

Create:

```text
.github/workflows/zap-scan.yml
```

Content:

```yaml
name: OWASP ZAP DAST Scan

on:
  push:
    branches:
      - main

  workflow_dispatch:

permissions:
  contents: read

jobs:
  zap_scan:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Run OWASP ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.14.0
        with:
          target: "http://54.92.155.35:3000"
          fail_action: false
          allow_issue_writing: false
          cmd_options: "-a"

      - name: Upload ZAP Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: zap-html-report
          path: report_html.html
```

---

## Push Workflow

```bash
git add .
git commit -m "Add OWASP ZAP workflow"
git push origin main
```

---

## Run Scan

Automatic:

```text
Push code to main branch
```

Manual:

```text
GitHub
→ Actions
→ OWASP ZAP DAST Scan
→ Run Workflow
```

---

## Download Report

Navigate:

```text
GitHub Repository
→ Actions
→ Successful Workflow Run
→ Artifacts
→ zap-html-report
```

Download:

```text
zap-html-report.zip
```

Extract:

```bash
unzip zap-html-report.zip
```

Open:

```bash
report_html.html
```

---

## Sample Findings

Medium Risk:

* Content Security Policy Header Not Set
* Cross-Domain Misconfiguration

Low Risk:

* COEP Header Missing
* COOP Header Missing
* Full Path Disclosure
* Timestamp Disclosure

---

## Troubleshooting

### Verify Application

```bash
curl http://54.92.155.35:3000
```

### Verify Container

```bash
docker ps
```

### Check Logs

```bash
docker logs juice-shop
```

### Verify Port

```bash
netstat -tulpn | grep 3000
```

or

```bash
ss -tulpn | grep 3000
```

### Security Group

Verify:

```text
Inbound Rules
-------------
22    0.0.0.0/0
3000  0.0.0.0/0
```

---

## CI/CD Security Flow

```text
Developer
    ↓
GitHub Push
    ↓
GitHub Actions
    ↓
OWASP ZAP Scan
    ↓
HTML Report
    ↓
Artifact Storage
    ↓
Developer Reviews Findings
    ↓
Remediation
    ↓
Re-Scan
    ↓
Production Deployment
```

---

## Technologies Used

* AWS EC2
* Docker
* GitHub
* GitHub Actions
* OWASP ZAP
* Juice Shop
* DevSecOps
* CI/CD


git add README.md && git commit -m "Add project documentation" && git push origin main
=============================================================================


Q: What is DAST?

Answer:

DAST (Dynamic Application Security Testing) scans a running application from outside, similar to an attacker, to identify vulnerabilities such as XSS, SQL Injection, insecure headers, information disclosure, and authentication issues.

Q: Why use OWASP ZAP?

Answer:

OWASP ZAP is an open-source DAST tool used to automatically discover and report web application security vulnerabilities during CI/CD pipelines.


Q: Difference between SAST and DAST?
SAST	               DAST
Scans                 Source Code	Scans Running Application
Before Deployment	     After Deployment
White Box Testing	      Black Box Testing
SonarQube, Checkmarx	    OWASP ZAP, Burp Suite

Q: Why upload report as artifact?

Answer:

Artifacts preserve scan reports after pipeline completion, allowing developers and security teams to download, review, audit, and remediate identified vulnerabilities.

