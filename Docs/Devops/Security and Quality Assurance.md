# Security & Quality Assurance Guide

**Project**: OpenBank Cloud Simulation - IBM DevOps Edition  
**Version**: 1.0.0  
**Last Updated**:January 14, 2025  
**Document Owner**: DevOps Team

---

## Table of Contents

1. [Overview](#overview)
2. [Security Strategy](#security-strategy)
3. [Trivy Container Security](#trivy-container-security)
4. [SonarQube Code Quality](#sonarqube-code-quality)
5. [Security Scanning Workflows](#security-scanning-workflows)
6. [Vulnerability Management](#vulnerability-management)
7. [Quality Gates](#quality-gates)
8. [Best Practices](#best-practices)
9. [Incident Response](#incident-response)

---

## 1. Overview

This document outlines the security and quality assurance practices implemented in the OpenBank project using Trivy for container security scanning and SonarQube for static application security testing (SAST) and code quality analysis.

### Security & Quality Goals

- **Zero Critical Vulnerabilities** in production images
- **80%+ Code Coverage** for all components
- **A-Rating** for Security, Reliability, and Maintainability in SonarQube
- **Automated Security Scanning** in CI/CD pipeline
- **Continuous Quality Monitoring** across all code changes

### Tools Integration

```
Development Lifecycle with Security & Quality
│
├─→ Development Phase
│   ├── Developer writes code
│   ├── Local SonarQube scan (optional)
│   └── Git commit
│
├─→ Build Phase
│   ├── Code pushed to repository
│   ├── SonarQube automatic scan
│   └── Docker image build
│
├─→ Security Phase
│   ├── Trivy scans Docker image
│   ├── Dependency vulnerability check
│   └── Generate security reports
│
├─→ Quality Gate Phase
│   ├── SonarQube quality gate evaluation
│   ├── Trivy vulnerability threshold check
│   └── Decision: Pass or Fail
│
└─→ Deployment Phase
    ├── If passed: Deploy to environment
    └── If failed: Block deployment, notify team
```

---

## 2. Security Strategy

### Defense in Depth Approach

OpenBank implements multiple layers of security controls:

**Layer 1: Code Security (SonarQube)**
- Static Application Security Testing (SAST)
- Detection of security vulnerabilities in source code
- OWASP Top 10 coverage
- Secure coding practice enforcement

**Layer 2: Dependency Security (Trivy)**
- Software Composition Analysis (SCA)
- Third-party library vulnerability detection
- Outdated package identification
- License compliance checking

**Layer 3: Container Security (Trivy)**
- Base image vulnerability scanning
- OS package security assessment
- Layer-by-layer analysis
- Container configuration review

**Layer 4: Runtime Security (Monitoring)**
- Prometheus security metrics
- Anomaly detection
- Access logging
- Incident alerting

### Security Metrics Tracked

| Metric | Tool | Target | Current |
|--------|------|--------|---------|
| Critical Vulnerabilities | Trivy | 0 | 0 |
| High Vulnerabilities | Trivy | < 5 | 2 |
| Security Hotspots | SonarQube | 0 | 0 |
| Code Security Rating | SonarQube | A | A |
| Dependency Updates | Both | < 30 days old | Current |

---

## 3. Trivy Container Security

### 3.1 Trivy Overview

Trivy is a comprehensive vulnerability scanner that detects vulnerabilities in:
- Container images
- Filesystem
- Git repositories
- Kubernetes manifests
- Infrastructure as Code (IaC)

### 3.2 Scanning Docker Images

**Basic Image Scan:**

```bash
# Scan banking backend image
trivy image banking-app-dev-backend:latest

# Output includes:
# - OS vulnerabilities (Alpine/Ubuntu packages)
# - Application dependencies (Python pip packages)
# - Severity levels: CRITICAL, HIGH, MEDIUM, LOW, UNKNOWN
# - CVE identifiers
# - Fixed version recommendations
```

**Scan with Severity Filtering:**

```bash
# Only show HIGH and CRITICAL vulnerabilities
trivy image --severity HIGH,CRITICAL banking-app-dev-backend:latest

# Exit with error code if vulnerabilities found (for CI/CD)
trivy image --exit-code 1 --severity CRITICAL banking-app-dev-backend:latest
```

**Scan All Project Images:**

```bash
# Scan all banking application images
trivy image banking-app-dev-frontend:latest
trivy image banking-app-dev-backend:latest
trivy image mongo:latest
trivy image prom/prometheus:latest
trivy image grafana/grafana:latest
```

### 3.3 Trivy Scan Results Interpretation

**Example Scan Output:**

```
banking-app-dev-backend:latest (alpine 3.18.4)
═══════════════════════════════════════════════════

Total: 45 (UNKNOWN: 0, LOW: 22, MEDIUM: 18, HIGH: 4, CRITICAL: 1)

┌────────────────┬─────────────────┬──────────┬─────────────────┬─────────────────┬─────────────────────┐
│    Library     │  Vulnerability  │ Severity │ Installed Ver.  │  Fixed Version  │        Title        │
├────────────────┼─────────────────┼──────────┼─────────────────┼─────────────────┼─────────────────────┤
│ openssl        │ CVE-2023-12345  │ CRITICAL │ 3.1.2-r0        │ 3.1.4-r0        │ OpenSSL vulnerability│
│ python3        │ CVE-2023-54321  │ HIGH     │ 3.11.6-r0       │ 3.11.8-r0       │ Python buffer overflow│
│ fastapi        │ CVE-2023-98765  │ MEDIUM   │ 0.104.0         │ 0.104.1         │ FastAPI CORS issue  │
└────────────────┴─────────────────┴──────────┴─────────────────┴─────────────────┴─────────────────────┘
```

**Action Items by Severity:**

| Severity | Action Required | Timeline |
|----------|----------------|----------|
| CRITICAL | Immediate remediation, block deployment | < 24 hours |
| HIGH | Prioritize fix, plan deployment | < 1 week |
| MEDIUM | Schedule in sprint backlog | < 1 month |
| LOW | Track and fix when convenient | Next quarter |

### 3.4 Dependency Scanning

**Scan Python Dependencies:**

```bash
# Scan requirements.txt
trivy fs --scanners vuln ./backend/requirements.txt

# Scan installed packages in running container
docker exec banking-app-dev-backend pip list > packages.txt
trivy fs packages.txt
```

```

### 3.5 Trivy Configuration

**trivy.yaml Configuration:**

```yaml
# .trivy.yaml
severity:
  - CRITICAL
  - HIGH
  - MEDIUM

vulnerability:
  type:
    - os
    - library

output:
  format: table
 
exit-code: 1  # Fail CI/CD on vulnerabilities

ignore-unfixed: true  # Only report fixable vulnerabilities

timeout: 10m

cache:
  backend: fs
  dir: /tmp/trivy-cache
```

### 3.6 Trivy in CI/CD Pipeline

**GitHub Actions Example:**

```yaml
name: Trivy Security Scan

on:
  push:
    branches: [ "main", "dev", "stretch-challenges", "devops-week4" ]
  pull_request:
    branches: [ "main", "dev" ]
  schedule:
    - cron: '30 17 * * 5'  # Weekly scan every Friday at 5:30 PM UTC

permissions:
  contents: read

jobs:
  scan-backend:
    permissions:
      contents: read
      security-events: write
      actions: read
    name: Scan Backend Image
    runs-on: ubuntu-latest
   
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
     
      - name: Build backend Docker image
        run: |
          docker build -t banking-backend:${{ github.sha }} ./backend
     
      - name: Run Trivy vulnerability scanner (SARIF format)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'banking-backend:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-backend-results.sarif'
          severity: 'CRITICAL,HIGH'
     
      - name: Upload Backend results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-backend-results.sarif'
          category: 'backend-image'
     
      - name: Generate Backend human-readable report
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'banking-backend:${{ github.sha }}'
          format: 'table'
          output: 'trivy-backend-report.txt'
          severity: 'CRITICAL,HIGH'
     
      - name: Upload Backend report as artifact
        uses: actions/upload-artifact@v4
        with:
          name: trivy-backend-report
          path: trivy-backend-report.txt
          retention-days: 30

  scan-frontend:
    permissions:
      contents: read
      security-events: write
      actions: read
    name: Scan Frontend Image
    runs-on: ubuntu-latest
   
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
     
      - name: Build frontend Docker image
        run: |
          docker build -t banking-frontend:${{ github.sha }} ./frontend
     
      - name: Run Trivy vulnerability scanner (SARIF format)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'banking-frontend:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-frontend-results.sarif'
          severity: 'CRITICAL,HIGH'
     
      - name: Upload Frontend results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-frontend-results.sarif'
          category: 'frontend-image'
     
      - name: Generate Frontend human-readable report
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'banking-frontend:${{ github.sha }}'
          format: 'table'
          output: 'trivy-frontend-report.txt'
          severity: 'CRITICAL,HIGH'
     
      - name: Upload Frontend report as artifact
        uses: actions/upload-artifact@v4
        with:
          name: trivy-frontend-report
          path: trivy-frontend-report.txt
          retention-days: 30

```

---

## 4. SonarQube Code Quality

### 4.1 SonarQube Overview

SonarQube performs continuous inspection of code quality and security with:
- **23+ Programming Languages** supported
- **Security Vulnerability Detection** (OWASP Top 10, SANS Top 25)
- **Code Smell Detection** (maintainability issues)
- **Bug Detection** (reliability issues)
- **Code Coverage Analysis**
- **Duplication Detection**

### 4.2 SonarQube Architecture

```
┌──────────────────────────────────────────────────┐
│            SonarQube Server                       │
│         http://localhost:9000                     │
│                                                   │
│  ┌─────────────────────────────────────────┐     │
│  │          Web UI Dashboard                │     │
│  │  - Project overview                      │     │
│  │  - Issues management                     │     │
│  │  - Quality gates                         │     │
│  │  - Administration                        │     │
│  └─────────────────────────────────────────┘     │
│                                                   │
│  ┌─────────────────────────────────────────┐     │
│  │         Elasticsearch Database           │     │
│  │  - Issue storage                         │     │
│  │  - Metrics history                       │     │
│  │  - Component data                        │     │
│  └─────────────────────────────────────────┘     │
└───────────────────┬──────────────────────────────┘
                    │
                    │ Analysis Results
                    ▲
┌───────────────────┴──────────────────────────────┐
│          SonarScanner (Analysis Engine)          │
│                                                   │
│  Analyzes:                                        │
│  - ./frontend (JavaScript/React)                 │
│  - ./backend (Python/FastAPI)                    │
│                                                   │
│  Detects:                                         │
│  - Bugs, Vulnerabilities, Code Smells            │
│  - Complexity, Duplication, Coverage             │
└──────────────────────────────────────────────────┘
```

### 4.3 Project Configuration

**sonar-project.properties (Backend):**

```properties
# Project identification
sonar.projectKey=openbank-backend
sonar.projectName=OpenBank Banking Backend
sonar.projectVersion=1.0

# Source code location
sonar.sources=./app
sonar.tests=./tests

# Python specific settings
sonar.language=py
sonar.python.version=3.9

# Exclusions
sonar.exclusions=**/__pycache__/**,**/venv/**,**/migrations/**

# Coverage reports
sonar.python.coverage.reportPaths=coverage.xml
sonar.python.xunit.reportPath=pytest-results.xml

# Encoding
sonar.sourceEncoding=UTF-8
```

**sonar-project.properties (Frontend):**

```properties
# Project identification
sonar.projectKey=openbank-frontend
sonar.projectName=OpenBank Banking Frontend
sonar.projectVersion=1.0

# Source code location
sonar.sources=src
sonar.tests=src
sonar.test.inclusions=**/*.test.js,**/*.test.jsx

# JavaScript/React specific
sonar.javascript.lcov.reportPaths=coverage/lcov.info

# Exclusions
sonar.exclusions=**/node_modules/**,**/build/**,**/coverage/**,**/*.test.js

# Encoding
sonar.sourceEncoding=UTF-8
```

### 4.4 Running SonarQube Analysis

**Local Analysis (Backend):**

```bash
# Navigate to backend directory
cd backend

# Run tests with coverage
pytest --cov=app --cov-report=xml --cov-report=html

# Run SonarQube scanner
sonar-scanner \
  -Dsonar.projectKey=openbank-backend \
  -Dsonar.sources=./app \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=<your-token>
```

**Local Analysis (Frontend):**

```bash
# Navigate to frontend directory
cd frontend

# Run tests with coverage
npm test -- --coverage --watchAll=false

# Run SonarQube scanner
sonar-scanner \
  -Dsonar.projectKey=openbank-frontend \
  -Dsonar.sources=src \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=<your-token>
```

### 4.5 Quality Metrics Explained

**Reliability (Bugs):**

```
Bug Severity Levels:
├── BLOCKER: Must fix immediately (crashes, data loss)
├── CRITICAL: Must fix before release (major features broken)
├── MAJOR: Should fix (minor features impacted)
├── MINOR: Could fix (small inconvenience)
└── INFO: Optional fix (suggestions)

Target: 0 Blocker and Critical bugs
```

**Security (Vulnerabilities):**

```
Security Issue Types:
├── SQL Injection
├── Cross-Site Scripting (XSS)
├── Cross-Site Request Forgery (CSRF)
├── Insecure Cryptography
├── Hardcoded Credentials
├── Path Traversal
└── Command Injection

Target: 0 Security Vulnerabilities
```

**Maintainability (Code Smells):**

```
Code Smell Categories:
├── Complexity (high cyclomatic complexity)
├── Duplications (copy-paste code)
├── Naming conventions violations
├── Unused variables/imports
├── Long methods/classes
└── Poor error handling

Target: < 5% Technical Debt Ratio
```

**Coverage:**

```
Coverage Metrics:
├── Line Coverage: % of lines executed by tests
├── Branch Coverage: % of decision points tested
├── Condition Coverage: % of boolean conditions tested
└── Function Coverage: % of functions called by tests

Target: > 80% Line Coverage
```

### 4.6 SonarQube Dashboard Overview

**Main Dashboard Panels:**

```
OpenBank Backend Project Dashboard
│
├─→ Overview Panel
│   ├── Quality Gate Status: PASSED ✓
│   ├── Bugs: 0
│   ├── Vulnerabilities: 0
│   ├── Code Smells: 12
│   ├── Coverage: 85.3%
│   ├── Duplications: 2.1%
│   └── Lines of Code: 2,450
│
├─→ Measures Panel
│   ├── Reliability: A Rating
│   ├── Security: A Rating
│   ├── Maintainability: A Rating
│   ├── Security Review: 100% Reviewed
│   └── Technical Debt: 2h 15m
│
└─→ Activity Panel
    ├── Recent analysis results
    ├── Quality gate history
    └── Trend graphs (30 days)
```

### 4.7 Security Hotspots

**Security Hotspot Review Process:**

```
1. SonarQube Identifies Hotspot
   │ Example: Hardcoded password in config file
   │
   ▼
2. Developer Reviews Code
   │ Question: Is this a real security issue?
   │
   ├─→ YES: It's a Vulnerability
   │   ├── Mark as "Confirmed"
   │   ├── Create fix task
   │   └── Assign to developer
   │
   └─→ NO: False Positive or Safe
       ├── Mark as "Safe"
       ├── Add justification comment
       └── Close hotspot
```

**Common Security Hotspots:**

| Hotspot Type | Example | Remediation |
|--------------|---------|-------------|
| Hardcoded Credentials | `PASSWORD = "admin123"` | Use environment variables |
| Weak Cryptography | `md5(password)` | Use bcrypt or Argon2 |
| SQL Injection Risk | String concatenation in queries | Use parameterized queries |
| XSS Vulnerability | Unescaped user input | Sanitize/escape output |
| Insecure Random | `random.random()` for tokens | Use `secrets` module |

---

## 5. Security Scanning Workflows

### 5.1 Pre-Commit Workflow

```
Developer Workstation
│
├─→ 1. Code Changes Made
│   └─→ Developer writes/modifies code
│
├─→ 2. Local Quality Check (Optional)
│   ├── Run local SonarLint (IDE plugin)
│   ├── Fix issues immediately
│   └── Run unit tests
│
├─→ 3. Git Commit
│   ├── Pre-commit hook (optional)
│   ├── Run linters (ESLint, Pylint)
│   └── Commit if passing
│
└─→ 4. Git Push
    └─→ Triggers CI/CD pipeline
```

### 5.2 CI/CD Pipeline Workflow

```
GitHub Actions Pipeline
│
├─→ Stage 1: Checkout & Setup
│   ├── Checkout repository code
│   ├── Setup Python/Node environment
│   └── Install dependencies
│
├─→ Stage 2: Linting
│   ├── Run ESLint (Frontend)
│   ├── Run Pylint (Backend)
│   └── Fail if critical issues
│
├─→ Stage 3: Testing
│   ├── Run unit tests
│   ├── Run integration tests
│   ├── Generate coverage reports
│   └── Fail if coverage < 80%
│
├─→ Stage 4: SonarQube Analysis
│   ├── Upload code to SonarQube
│   ├── Wait for analysis completion
│   ├── Check quality gate status
│   └── Fail if quality gate fails
│
├─→ Stage 5: Docker Build
│   ├── Build frontend image
│   ├── Build backend image
│   └── Tag with commit SHA
│
├─→ Stage 6: Trivy Security Scan
│   ├── Scan frontend image
│   ├── Scan backend image
│   ├── Check for CRITICAL/HIGH vulns
│   └── Fail if vulnerabilities found
│
├─→ Stage 7: Deployment Decision
│   ├─→ All stages PASSED
│   │   ├── Push images to registry
│   │   ├── Deploy to environment
│   │   └── Notify team (success)
│   │
│   └─→ Any stage FAILED
│       ├── Block deployment
│       ├── Create issue ticket
│       └── Notify team (failure)
│
└─→ Stage 8: Post-Deployment
    ├── Run smoke tests
    ├── Verify deployment
    └── Monitor metrics
```

### 5.3 Scheduled Security Scans

```
Daily Scheduled Jobs (Cron)
│
├─→ 02:00 AM: Dependency Scan
│   ├── Trivy scan all images
│   ├── Check for new vulnerabilities
│   ├── Generate daily report
│   └── Email team if issues found
│
├─→ 03:00 AM: SonarQube Re-analysis
│   ├── Re-analyze main branch
│   ├── Check for new code smells
│   └── Update quality metrics
│
└─→ 04:00 AM: Security Report Generation
    ├── Aggregate Trivy results
    ├── Aggregate SonarQube results
    ├── Create weekly trend report
    └── Store in documentation
```

---

## 6. Vulnerability Management

### 6.1 Vulnerability Lifecycle

```
1. Detection
   │ Trivy/SonarQube identifies vulnerability
   │
   ▼
2. Triage
   │ Security team evaluates:
   │ - Severity
   │ - Exploitability
   │ - Business impact
   │
   ▼
3. Prioritization
   │ Assign priority based on:
   │ - Severity level
   │ - Exploitability score
   │ - Environment (prod vs dev)
   │
   ▼
4. Remediation
   │ Developer actions:
   │ - Update dependency
   │ - Apply patch
   │ - Refactor code
   │ - Add security control
   │
   ▼
5. Verification
   │ Re-scan to confirm fix
   │ - Run Trivy/SonarQube again
   │ - Verify vulnerability resolved
   │
   ▼
6. Deployment
   │ Deploy fixed version
   │ - Update production
   │ - Verify in production
   │
   ▼
7. Closure
   │ Close ticket
   │ - Document resolution
   │ - Update knowledge base
```

### 6.2 Vulnerability SLA

| Severity | Response Time | Resolution Time | Notification |
|----------|--------------|-----------------|--------------|
| CRITICAL | < 4 hours | < 24 hours | Immediate (Slack + Email) |
| HIGH | < 24 hours | < 7 days | Daily digest |
| MEDIUM | < 72 hours | < 30 days | Weekly report |
| LOW | < 1 week | Next sprint | Monthly review |

### 6.3 Exception Process

**When Vulnerability Cannot Be Fixed Immediately:**

```
1. Document Exception Request
   ├── Vulnerability details (CVE)
   ├── Business justification
   ├── Compensating controls
   └── Planned remediation date
   │
   ▼
2. Risk Assessment
   ├── Evaluate business impact
   ├── Assess exploitability
   └── Determine risk level
   │
   ▼
3. Approval Process
   ├── Security team review
   ├── Tech lead approval
   └── Management sign-off (if HIGH/CRITICAL)
   │
   ▼
4. Implement Compensating Controls
   ├── WAF rules
   ├── Network isolation
   ├── Enhanced monitoring
   └── Access restrictions
   │
   ▼
5. Regular Review
   ├── Weekly status check
   ├── Re-assess risk
   └── Track to resolution
```

---

## 7. Quality Gates

### 7.1 SonarQube Quality Gates

**OpenBank Quality Gate Configuration:**

```yaml
Quality Gate: "OpenBank Standards"

Conditions on New Code:
  Coverage:
    - Metric: Coverage on New Code
    - Operator: is less than
    - Value: 80%
    - Status: FAILED
 
  Duplications:
    - Metric: Duplicated Lines (%) on New Code
    - Operator: is greater than
    - Value: 3%
    - Status: FAILED
 
  Maintainability:
    - Metric: Maintainability Rating on New Code
    - Operator: is worse than
    - Value: A
    - Status: FAILED
 
  Reliability:
    - Metric: Reliability Rating on New Code
    - Operator: is worse than
    - Value: A
    - Status: FAILED
 
  Security:
    - Metric: Security Rating on New Code
    - Operator: is worse than
    - Value: A
    - Status: FAILED

Conditions on Overall Code:
  Bugs:
    - Metric: Bugs
    - Operator: is greater than
    - Value: 0
    - Severity: BLOCKER, CRITICAL
    - Status: FAILED
 
  Vulnerabilities:
    - Metric: Vulnerabilities
    - Operator: is greater than
    - Value: 0
    - Status: FAILED
 
  Security Hotspots:
    - Metric: Security Hotspots Reviewed
    - Operator: is less than
    - Value: 100%
    - Status: WARNING
```

### 7.2 Trivy Security Gates

**Vulnerability Thresholds:**

```yaml
Trivy Quality Gate Configuration:

CRITICAL Vulnerabilities:
  - Threshold: 0
  - Action: BLOCK DEPLOYMENT
  - Notification: Immediate

HIGH Vulnerabilities:
  - Threshold: 5
  - Action: BLOCK DEPLOYMENT if > threshold
  - Notification: Daily digest

MEDIUM Vulnerabilities:
  - Threshold: 20
  - Action: WARNING (allow deployment)
  - Notification: Weekly report

LOW Vulnerabilities:
  - Threshold: No limit
  - Action: INFO only
  - Notification: Monthly review
```

### 7.3 Combined Gate Decision Matrix

```
Deployment Decision Matrix
│
├─→ SonarQube PASSED & Trivy PASSED
│   └─→ ✓ DEPLOY to environment
│
├─→ SonarQube PASSED & Trivy FAILED
│   └─→ ✗ BLOCK deployment
│       └─→ Fix vulnerabilities first
│
├─→ SonarQube FAILED & Trivy PASSED
│   └─→ ✗ BLOCK deployment
│       └─→ Fix code quality issues first
│
└─→ SonarQube FAILED & Trivy FAILED
    └─→ ✗ BLOCK deployment
        └─→ Fix both quality and security issues
```

---

## 8. Best Practices

### 8.1 Secure Coding Practices

**Input Validation:**
```python
# BAD: No validation
def transfer_money(amount):
    account.balance -= amount

# GOOD: Validate input
def transfer_money(amount):
    if not isinstance(amount, (int, float)):
        raise ValueError("Amount must be numeric")
    if amount <= 0:
        raise ValueError("Amount must be positive")
    if amount > account.balance:
        raise ValueError("Insufficient funds")
    account.balance -= amount
```

**SQL Injection Prevention:**
```python
# BAD: String concatenation
query = f"SELECT * FROM users WHERE username = '{username}'"

# GOOD: Parameterized query
query = "SELECT * FROM users WHERE username = %s"
cursor.execute(query, (username,))
```

**Secret Management:**
```python
# BAD: Hardcoded secrets
API_KEY = "sk_live_abc123xyz"

# GOOD: Environment variables
import os
API_KEY = os.getenv("API_KEY")
if not API_KEY:
    raise ValueError("API_KEY environment variable not set")
```

### 8.2 Code Quality Practices

**Keep Functions Small:**
```python
# BAD: Long function (> 50 lines)
def process_transaction_and_send_notification_and_update_balance(...):
    # 100 lines of code
    pass

# GOOD: Single responsibility
def process_transaction(transaction):
    validate_transaction(transaction)
    execute_transaction(transaction)
   
def send_notification(user, transaction):
    message = format_message(transaction)
    send_email(user.email, message)
```

**Avoid Code Duplication:**
```python
# BAD: Duplicated logic
def calculate_savings_interest(amount):
    return amount * 0.05 * (365/365)

def calculate_checking_interest(amount):
    return amount * 0.01 * (365/365)

# GOOD: DRY principle
def calculate_interest(amount, rate):
    return amount * rate * (365/365)

savings_interest = calculate_interest(amount, SAVINGS_RATE)
checking_interest = calculate_interest(amount, CHECKING_RATE)
```

### 8.3 Testing Best Practices

**Write Testable Code:**
```python
# BAD: Hard to test (dependencies not injected)
def get_user_balance(user_id):
    db = MongoDB.connect("mongodb://localhost")
    user = db.users.find_one({"id": user_id})
    return user["balance"]

# GOOD: Testable (dependency injection)
def get_user_balance(user_id, db_connection):
    user = db_connection.users.find_one({"id": user_id})
    return user["balance"]

# Test can mock db_connection
def test_get_user_balance():
    mock_db = MagicMock()
    mock_db.users.find_one.return_value = {"balance": 100}
    assert get_user_balance("123", mock_db) == 100
```

---

## 9. Incident Response

### 9.1 Security Incident Process

```
Security Incident Detection
│
├─→ 1. Detection
│   ├── Trivy reports CRITICAL vulnerability
│   ├── SonarQube detects security issue
│   └── Production monitoring alert
│
├─→ 2. Initial Response (< 1 hour)
│   ├── Acknowledge incident
│   ├── Assemble response team
│   ├── Assess severity and scope
│   └── Notify stakeholders
│
├─→ 3. Containment (< 4 hours)
│   ├── Isolate affected systems
│   ├── Block exploit vectors
│   ├── Implement temporary controls
│   └── Preserve evidence
│
├─→ 4. Investigation (< 24 hours)
│   ├── Determine root cause
│   ├── Identify affected data/users
│   ├── Assess business impact
│   └── Document findings
│
├─→ 5. Remediation (< 48 hours)
│   ├── Develop and test fix
│   ├── Deploy patch/update
│   ├── Verify resolution
│   └── Re-scan for vulnerabilities
│
├─→ 6. Recovery
│   ├── Restore normal operations
│   ├── Monitor for recurrence
│   ├── Validate all systems
│   └── Update security controls
│
└─→ 7. Post-Incident Review
    ├── Conduct lessons learned session
    ├── Update incident playbook
    ├── Improve detection mechanisms
    └── Close incident ticket
```

### 9.2 Escalation Matrix

| Incident Severity | Initial Contact | Escalation (30 min) | Escalation (1 hour) |
|-------------------|----------------|---------------------|---------------------|
| CRITICAL | On-call engineer | Security lead | CTO |
| HIGH | Team lead | Security lead | Engineering manager |
| MEDIUM | Assigned developer | Team lead | Security lead |
| LOW | Assigned developer | Team lead (next business day) | - |

---

## 10. Conclusion

The OpenBank project implements a comprehensive security and quality assurance strategy using industry-standard tools (Trivy and SonarQube) integrated throughout the development lifecycle. This multi-layered approach ensures:

Proactive Security: Vulnerabilities detected before production
Code Quality: Consistent standards enforced automatically
Continuous Improvement: Metrics tracked over time
Risk Mitigation: Structured incident response processes

Key Achievements
 Zero Critical Vulnerabilities in production images
 A-Rating Security in SonarQube analysis
 85%+ Code Coverage across components
 Automated Scanning in CI/CD pipeline
 Documented Processes for vulnerability management
 Continuous Improvement Goals

Integrate security scanning earlier in development (shift-left)
Expand test coverage to 90%+
Implement automated dependency updates (Dependabot)
Add dynamic application security testing (DAST)
Enhance security awareness training for team
