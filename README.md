# 🚀 DevOps Project 008 — End-to-End CI/CD Pipeline for Android Apps with GitHub Actions

<p align="center">
  <img src="https://imgur.com/XNUS0pA.png" alt="Android CI/CD Banner" width="800"/>
</p>

<p align="center">
  <a href="https://github.com/hmurafique/DevOps-Project-008/actions/workflows/android-ci-cd.yml">
    <img src="https://github.com/hmurafique/DevOps-Project-008/actions/workflows/android-ci-cd.yml/badge.svg" alt="Android CI and CD"/>
  </a>
  <img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white" alt="GitHub Actions"/>
  <img src="https://img.shields.io/badge/Android-3DDC84?style=flat&logo=android&logoColor=white" alt="Android"/>
  <img src="https://img.shields.io/badge/Gradle-02303A?style=flat&logo=gradle&logoColor=white" alt="Gradle"/>
  <img src="https://img.shields.io/badge/SonarQube-4E9BCD?style=flat&logo=sonarqube&logoColor=white" alt="SonarQube"/>
  <img src="https://img.shields.io/badge/JFrog-41BF47?style=flat&logo=jfrog&logoColor=white" alt="JFrog"/>
  <img src="https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazonaws&logoColor=white" alt="AWS"/>
  <img src="https://img.shields.io/badge/Java-21-ED8B00?style=flat&logo=openjdk&logoColor=white" alt="Java 21"/>
  <img src="https://img.shields.io/badge/Kotlin-2.0.0-7F52FF?style=flat&logo=kotlin&logoColor=white" alt="Kotlin"/>
</p>

---

## 📌 Project Overview

This project demonstrates a fully automated **End-to-End CI/CD Pipeline** for an Android application using **GitHub Actions**. The pipeline automates building, testing, code quality analysis, and artifact deployment — allowing developers to focus on writing code while automation handles the rest.

### ✅ Key Features

- **Automated Builds** — Compile Android projects with Gradle 8.13
- **Continuous Integration** — Run lint checks and unit tests on every push
- **Code Quality** — SonarQube integration for static analysis
- **Artifact Management** — Store APK files as GitHub artifacts
- **Secure Deployments** — Upload APK to JFrog Artifactory
- **Cache Cleanup** — Automatically delete caches and old artifacts
- **Multi-branch Support** — Triggers on `main`, `qa`, and `develop` branches

---

## 🏗️ Architecture

```
Developer Push (main/qa/develop)
         │
         ▼
  GitHub Actions Trigger
         │
    ┌────┴─────────────────────────┐
    │         BUILD JOB            │
    │  ① Checkout Code             │
    │  ② Setup JDK 21              │
    │  ③ Generate Debug Keystore   │
    │  ④ Gradle Clean              │
    │  ⑤ Lint Check                │
    │  ⑥ AssembleDebug (APK)       │
    │  ⑦ Unit Tests                │
    │  ⑧ SonarQube Analysis        │
    │  ⑨ Upload APK Artifact       │
    └────────────────────────────┬─┘
                                 │
                    ┌────────────▼──────────────┐
                    │        DEPLOY JOB          │
                    │  ① Download APK Artifact   │
                    │  ② Setup JFrog CLI         │
                    │  ③ Upload to Artifactory   │
                    └───────────────────────────┘
                                 │
                    ┌────────────▼──────────────┐
                    │      CACHE CLEANUP         │
                    │  Auto-triggered on         │
                    │  workflow completion        │
                    └───────────────────────────┘
```

---

## 🛠️ Tools & Versions

| Tool | Version | Purpose |
|------|---------|---------|
| GitHub Actions | Latest | CI/CD Platform |
| Android Gradle Plugin | 8.8.0 | Android Build System |
| Gradle | 8.13 | Build Automation |
| Java (JDK) | 21 (Temurin) | Runtime |
| Kotlin | 2.0.0 | Programming Language |
| SonarQube Community | 26.6.0.123539 | Code Quality |
| JFrog Artifactory OSS | Latest | Artifact Repository |
| AWS EC2 | Ubuntu 22.04 | Server Infrastructure |

---

## 📁 Repository Structure

```
DevOps-Project-008/
├── .github/
│   └── workflows/
│       ├── android-ci-cd.yml       # Main CI/CD pipeline
│       ├── clear-cache.yml         # Cache cleanup workflow
│       └── cleanup-artifacts.yml  # Artifact cleanup (cron)
├── app/
│   ├── src/
│   │   ├── main/                   # App source code
│   │   └── test/                   # Unit tests
│   └── build.gradle.kts            # App-level Gradle config
├── gradle/
│   ├── libs.versions.toml          # Dependency versions catalog
│   └── wrapper/
│       └── gradle-wrapper.properties
├── build.gradle.kts                # Root Gradle config (SonarQube plugin)
├── settings.gradle.kts
├── gradlew
└── gradlew.bat
```

---

## 🚀 Step-by-Step Implementation Guide

### Prerequisites

- AWS Account (for EC2 instances)
- GitHub Account
- Android app source code with Gradle build system

---

### PHASE 1 — SonarQube Server Setup (EC2)

#### 1.1 — Launch EC2 Instance

| Setting | Value |
|---------|-------|
| Name | `sonarqube-server` |
| AMI | Ubuntu 22.04 LTS |
| Instance Type | `t3.medium` |
| Storage | 20 GB |
| Security Group Ports | 22 (SSH), 9000 (SonarQube) |

#### 1.2 — Install SonarQube

```bash
# SSH into EC2
ssh -i your-key.pem ubuntu@<EC2-PUBLIC-IP>

# Update system
sudo apt update && sudo apt upgrade -y

# Install Java 21 JDK (required by SonarQube 26.x)
sudo apt install -y openjdk-21-jdk
java -version
# Expected: openjdk version "21.x.x"

# Set system limits (SonarQube requirement)
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w fs.file-max=65536
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
echo "fs.file-max=65536" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Download SonarQube Community Build 26.6.0
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-26.6.0.123539.zip
sudo apt install -y unzip
sudo unzip sonarqube-26.6.0.123539.zip
sudo mv sonarqube-26.6.0.123539 sonarqube
sudo rm sonarqube-26.6.0.123539.zip

# Create dedicated user
sudo adduser --system --no-create-home --group --disabled-login sonarqube
sudo chown -R sonarqube:sonarqube /opt/sonarqube

# Start SonarQube
sudo -u sonarqube /opt/sonarqube/bin/linux-x86-64/sonar.sh start

# Check status (wait 1-2 minutes)
sudo -u sonarqube /opt/sonarqube/bin/linux-x86-64/sonar.sh status
# Expected: SonarQube is running (PID XXXXX)
```

#### 1.3 — Configure SonarQube Project

1. Open browser: `http://<EC2-PUBLIC-IP>:9000`
2. Login: `admin` / `admin` → set new password
3. Create Project → **Create a local project**
   - Project Key: `Android-CICD`
   - Display Name: `Android-CICD`
   - Main branch: `main`
4. Generate Token: `My Account → Security → Generate Token`
   - Name: `github-actions-token`
   - Type: `Global Analysis Token`
   - Expiry: `No expiration`
   - **Copy the token — shown only once!**

---

### PHASE 2 — Android App Configuration

#### 2.1 — Add SonarQube Plugin to root `build.gradle.kts`

```kotlin
plugins {
    id("org.sonarqube") version "7.3.0.8198"
    // ... other plugins
}
```

#### 2.2 — Configure lint in `app/build.gradle.kts`

```kotlin
android {
    lint {
        abortOnError = false
        checkReleaseBuilds = false
        disable += "CoroutineCreationDuringComposition"
    }
    // ...
}
```

---

### PHASE 3 — GitHub Secrets Configuration

Go to: `Repository → Settings → Secrets and variables → Actions`

| Secret Name | Value |
|-------------|-------|
| `SONAR_TOKEN` | Token generated from SonarQube |
| `SONAR_HOST_URL` | `http://<sonarqube-ec2-ip>:9000` |
| `JF_URL` | `http://<jfrog-ec2-ip>:8082/artifactory` |
| `JF_ACCESS_TOKEN` | Token from JFrog Admin |
| `AWS_ACCESS_KEY_ID` | IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret key |
| `JFROG_SG_ID` | JFrog EC2 Security Group ID |

---

### PHASE 4 — GitHub Actions Workflows

Three workflow files are configured:

**1. `.github/workflows/android-ci-cd.yml`** — Main pipeline (build + deploy)

**2. `.github/workflows/clear-cache.yml`** — Clears GitHub Actions cache after each run

**3. `.github/workflows/cleanup-artifacts.yml`** — Deletes old artifacts every hour (cron)

---

### PHASE 5 — Pipeline Execution

Push code to trigger the pipeline:

```bash
git add .
git commit -m "feat: your commit message"
git push origin main
```

Monitor at: `https://github.com/hmurafique/DevOps-Project-008/actions`

---

## 📊 SonarQube Results

| Metric | Result |
|--------|--------|
| Quality Gate | ✅ Passed |
| Lines of Code | 2.9k |
| Languages | Kotlin, XML |
| Security | C (1 issue) |
| Reliability | C (14 issues) |
| Maintainability | A (28 issues) |
| Coverage | 0.0% |
| Duplications | 0.0% |

---

## ⚠️ Common Issues & Fixes

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| `validateSigningDebug FAILED` — BouncyCastle error | AGP 8.4.0 + JDK 21 incompatibility | Upgrade AGP to 8.8.0, use `-x validateSigningDebug` flag |
| `lintAnalyzeDebug FAILED` — CoroutineCreationDuringComposition | AGP 8.8 lint bug | Add `disable += "CoroutineCreationDuringComposition"` in lint config |
| `packagingOptions` deprecated warning | Old AGP syntax | Rename to `packaging {}` |
| SonarQube HTTP 401 Unauthorized | Token expired | Regenerate token in SonarQube → update GitHub Secret |
| `debug.keystore` not found | Missing keystore on CI runner | Generate keystore in workflow before build step |
| Gradle `vm.max_map_count too low` | SonarQube EC2 requirement | Run `sudo sysctl -w vm.max_map_count=262144` |

---

## 🧹 Cleanup

```bash
# Terminate EC2 instances
# AWS Console → EC2 → Select sonarqube-server → Terminate

# Delete GitHub Secrets (optional)
# Repository → Settings → Secrets → Delete

# Delete workflow artifacts
# Repository → Actions → Artifacts → Delete
```

---

## 📚 Reference

- [Original Project Reference](https://github.com/NotHarshhaa/DevOps-Projects/tree/master/DevOps-Project-14)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [SonarQube Community Build Docs](https://docs.sonarsource.com/sonarqube-community-build)
- [JFrog CLI Documentation](https://docs.jfrog-applications.jfrog.io/jfrog-applications/jfrog-cli)
- [Android Gradle Plugin Release Notes](https://developer.android.com/build/releases/about-agp)

---

## 🛠️ Author

**Hafiz Muhammad Umar Rafique**
DevOps & Cloud Engineer

[![GitHub](https://img.shields.io/badge/GitHub-hmurafique-181717?style=flat&logo=github&logoColor=white)](https://github.com/hmurafique)
[![DockerHub](https://img.shields.io/badge/DockerHub-hmurafique93-2496ED?style=flat&logo=docker&logoColor=white)](https://hub.docker.com/u/hmurafique93)

---

> **Note:** This project is part of a hands-on DevOps learning series based on [NotHarshhaa/DevOps-Projects](https://github.com/NotHarshhaa/DevOps-Projects). All implementation, fixes, and configurations are done independently from scratch.
