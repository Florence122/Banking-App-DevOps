# OpenBank Cloud Simulation - IBM DevOps Edition

**A Complete 4-Week DevOps Implementation Project**

[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker)](https://www.docker.com/)
[![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-E6522C?logo=prometheus)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Visualization-Grafana-F46800?logo=grafana)](https://grafana.com/)
[![Status](https://img.shields.io/badge/Status-Complete-success)]()

---

##  Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Key Features](#key-features)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Monitoring & Observability](#monitoring--observability)
- [DevOps Practices](#devops-practices)
- [Documentation](#documentation)
- [Team](#team)
- [Acknowledgments](#acknowledgments)

---

##  Project Overview

OpenBank is a comprehensive banking application developed as part of the IBM DevOps Professional Certificate program. This project demonstrates end-to-end DevOps practices including containerization, CI/CD pipelines, monitoring, and Site Reliability Engineering (SRE) principles.

**Project Duration**: 4 Weeks (November 20 - December 18, 2025)  
**Team Size**: 4 Members  
**Deployment**: Docker-based microservices architecture

### Business Context

A modern banking application featuring:
- User account management
- Transaction processing
- Balance inquiries
- Secure authentication
- Real-time monitoring and alerting

---

##  Architecture

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     Load Balancer / Nginx                    │
└───────────────────────┬─────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   Frontend   │ │   Backend    │ │  Monitoring  │
│   (React)    │ │  (FastAPI)   │ │   Stack      │
│   Port 3000  │ │  Port 8000   │ │              │
└──────┬───────┘ └──────┬───────┘ └──────┬───────┘
       │                │                │
       │         ┌──────┴─────┐         │
       │         ▼            ▼         ▼
       │   ┌──────────┐ ┌──────────────────┐
       └──►│ MongoDB  │ │  Prometheus      │
           │ Database │ │  + Grafana       │
           │Port 27017│ │  + Node Exporter │
           └──────────┘ │  + cAdvisor      │
                        └──────────────────┘
```

### Monitoring Architecture

```
User Interface Layer
    ↓
Grafana Dashboard (Port 3001)
    ↓
Prometheus Time-Series DB (Port 9090)
    ↓
┌────────────┬─────────────┬──────────────┐
│            │             │              │
Node Exporter  cAdvisor    Application
(System)    (Containers)   Metrics
```

---

##  Technologies Used

### Application Stack
- **Frontend**: React.js, HTML5, CSS3, JavaScript
- **Backend**: Python FastAPI, RESTful APIs
- **Database**: MongoDB (NoSQL)
- **Authentication**: JWT tokens

### DevOps Tools
- **Containerization**: Docker, Docker Compose
- **Orchestration**: Kubernetes (planned)
- **Monitoring**: Prometheus, Grafana, Node Exporter, cAdvisor
- **CI/CD**: GitHub Actions (planned)
- **Version Control**: Git, GitHub

### Infrastructure
- **Development**: Docker Desktop
- **Networking**: Docker Bridge Networks
- **Storage**: Docker Volumes (persistent data)

---

##  Key Features

### Application Features
-  User registration and authentication
-  Account balance management
-  Transaction history tracking
-  Secure API endpoints
-  Responsive web interface

### DevOps Features
-  Containerized microservices architecture
-  Real-time monitoring and alerting
-  Automated metrics collection
-  Performance baseline establishment
-  Service Level Objectives (SLOs)
-  Infrastructure as Code (IaC)

### Monitoring Capabilities
-  System resource monitoring (CPU, Memory, Disk, Network)
-  Container health tracking
-  Request rate and latency metrics
-  Service availability monitoring
-  Custom Grafana dashboards

---

##  Project Structure

```
banking-application/
├── frontend/                  # React frontend application
│   ├── src/
│   ├── public/
│   ├── Dockerfile
│   └── package.json
│
├── backend/                   # FastAPI backend service
│   ├── app/
│   ├── requirements.txt
│   └── Dockerfile
│
├── monitoring/                # Monitoring stack configuration
│   ├── prometheus/
│   │   └── prometheus.yml    # Prometheus scrape configs
│   ├── grafana/
│   │   └── dashboards/       # Grafana dashboard JSONs
│   └── docker-compose.monitoring.yml
│
├── docs/                      # Documentation
│   ├── Metrics Report.md
│   ├── SECURITY GUIDE.md
│   ├── ARCHITECTURE GUIDE.md
│   ├── FINAL WEEK_SUMMARY.md
│   ├── DEVOPS_METRICS_REPORT.md
│   ├── SLO_DOCUMENT.md
│   └── screenshots/
│
├── docker-compose.yml         # Main application compose file
└── README.md                  # This file
```

---

##  Getting Started

### Prerequisites

- Docker Desktop installed (version 20.10+)
- Docker Compose (version 2.0+)
- 4GB RAM minimum
- 10GB free disk space

### Quick Start

1. **Clone the Repository**
```bash
git clone https://github.com/Sekani-27/Banking-App-DevOps.git
cd banking-application
```

2. **Start the Application**
```bash
docker-compose up -d
```

3. **Start Monitoring Stack**
```bash
docker-compose -f docker-compose.monitoring.yml up -d
```

4. **Verify All Services**
```bash
docker ps
```

### Access Points

| Service | URL | Credentials |
|---------|-----|-------------|
| Frontend | http://localhost:3000 | - |
| Backend API | http://localhost:8000 | - |
| Prometheus | http://localhost:9090 | - |
| Grafana | http://localhost:3001 | admin / admin123 |
| MongoDB | localhost:27017 | - |

### Health Checks

```bash
# Check frontend
curl http://localhost:3000

# Check backend
curl http://localhost:8000/health

# Check Prometheus
curl http://localhost:9090/-/healthy

# Check Grafana
curl http://localhost:3001/api/health
```

---

##  Monitoring & Observability

### Grafana Dashboards

We've created a comprehensive monitoring dashboard with 5 key panels:

1. **HTTP Requests per Second** - Application traffic monitoring
2. **Service Status** - Real-time health checks
3. **Memory Usage** - Application memory consumption
4. **CPU Usage** - Processor utilization tracking
5. **Container Health** - Individual container metrics

### Key Metrics Tracked

- **System Metrics**: CPU, Memory, Disk I/O, Network
- **Container Metrics**: Resource usage per container
- **Application Metrics**: Request rates, response times
- **Database Metrics**: Connection pool, query performance (planned)

### Performance Baseline

Based on 6-hour monitoring window (Dec 18, 2025):

| Metric | Value | Status |
|--------|-------|--------|
| System Uptime | 100% |  Excellent |
| Avg Response Time | 250ms |  Target Met |
| CPU Utilization | 2.5% |  Optimal |
| Memory Usage | 500MB |  Stable |
| Request Success Rate | 97.5% |  Good |

---

##  DevOps Practices

### Week-by-Week Implementation

**Week 1: Agile Planning & Code Foundation**
- User story creation and sprint planning
- Repository setup with branching strategy
- Initial codebase development

**Week 2: Build, Test & Containerize**
- Dockerfile creation for all services
- Docker Compose orchestration
- Container networking configuration

**Week 3: CI/CD & Deployment Automation**
- GitHub Actions workflow setup (planned)
- Automated testing pipeline
- Deployment automation

**Week 4: Monitoring, SRE & Observability**
- Prometheus and Grafana deployment
- Dashboard creation and metric collection
- SLO definition and baseline establishment

### CI/CD Pipeline (Planned)

```
Code Push → Build → Test → Container Build → Deploy → Monitor
    │         │      │           │             │         │
    └─────────┴──────┴───────────┴─────────────┴─────────┘
              Automated DevOps Pipeline
```

---

##  Documentation

Comprehensive documentation available in `/docs`:

### Project Documentation
- [Architecture Guide](docs/ARCHITECTURE.md) - Complete system design and technical details
- [Security & Quality Assurance](docs/SECURITY_QUALITY.md) - Trivy and SonarQube implementation
- [Week 4 Summary](docs/WEEK4_SUMMARY.md) - Final week deliverables and project completion

### Technical Reports
- [DevOps Metrics Report](docs/DEVOPS_METRICS_REPORT.md) - Performance analysis and baseline metrics
- [SLO Document](docs/SLO_DOCUMENT.md) - Service Level Objectives and SRE practices

### Visual Documentation
- Screenshots of Grafana dashboards, running containers, and system monitoring
- Architecture diagrams and workflow visualizations

---

##  Team

**DevOps Team Members:**
- **Ntando** - Infrastructure & Configuration Management
- **Kagiso** - Monitoring & Dashboard Development
- **Florence** - Container Orchestration & Deployment
- **Tumelo** - SRE & Documentation

**Collaboration Tools:**
- GitHub for version control
- Daily standups for progress tracking
- Pair programming for complex configurations
- Code reviews for quality assurance

---

##  Key Achievements

### Technical Accomplishments
-  100% system uptime during monitoring period
-  Zero unplanned container restarts
-  97.5% request success rate
-  Sub-500ms response times (P95)
-  Efficient resource utilization (2.5% CPU)

### DevOps Maturity
-  Infrastructure as Code implementation
-  Automated monitoring and alerting foundation
-  Comprehensive documentation
-  Reproducible deployment process
-  Baseline performance metrics established

---

##  IBM DevOps Alignment

This project aligns with IBM DevOps Professional Certificate curriculum:

| Course | Implementation |
|--------|----------------|
| Introduction to DevOps | Applied DevOps culture and practices |
| Introduction to Agile | Implemented sprint planning and user stories |
| Application Security & Monitoring | Deployed Prometheus and Grafana |
| Continuous Integration & Delivery | Planned CI/CD pipeline architecture |
| Container & Kubernetes | Docker containerization and orchestration |

---

##  Future Enhancements

### Short-term (Next Sprint)
- [ ] Implement application-level metrics instrumentation
- [ ] Configure Prometheus alerting rules
- [ ] Add MongoDB metrics exporter
- [ ] Create additional Grafana dashboards

### Long-term (Production Readiness)
- [ ] Deploy to cloud platform (IBM Cloud/AWS)
- [ ] Implement distributed tracing with Jaeger
- [ ] Add log aggregation (ELK stack)
- [ ] Set up automated backup and recovery
- [ ] Implement chaos engineering tests

---

##  License

This project is part of an educational program and is intended for portfolio demonstration purposes.

---

##  Acknowledgments

- **IBM Skills Network** - DevOps Professional Certificate Program
- **Prometheus Community** - Monitoring tools and documentation
- **Grafana Labs** - Visualization platform
- **Docker** - Containerization technology

---

##  Contact

For questions or collaboration opportunities:

- **GitHub**: [@Sekani-27](https://github.com/Sekani-27/Banking-App-DevOps)
- **Project Repository**: https://github.com/Sekani-27/Banking-App-DevOps

---

**Project Status**:  Complete (4-Week Implementation)  
**Last Updated**: January 14, 2026 
**Version**: 1.0.0

---

