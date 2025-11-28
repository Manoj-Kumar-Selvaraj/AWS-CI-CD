# 🔥 BUILD SPEC  (CodeBuild)

# 🔥 APP SPEC  (CodeDeploy)

We will explain:

```
Every keyword
Every section
All syntaxes
Advanced patterns
Examples for Java, Node, Python, Docker, ECS, Lambda
Debugging, best practices + interview answers
```

By the end — you will MASTER both files better than 90% DevOps engineers.

---

# 🟥 PART–1 → `buildspec.yml` (FULL BREAKDOWN)

This file controls CodeBuild execution — like a **Jenkinsfile inside AWS**.

---

### 📌 Top Level Structure

```
version:
env:
phases:
artifacts:
cache:
reports:
```

Let’s break each one in detail 🔽

---

## 🔥 1) version

```yaml
version: 0.2   # Always recommended
```

Version 0.2 = latest feature support
Version 0.1 = old, rarely used today

---

## 🔥 2) env (VERY IMPORTANT — Interview Topic)

```yaml
env:
  variables:
    STAGE: "dev"                     # simple environment variable
    APP_NAME: "myapp"

  parameter-store:                   # Pull secure values from SSM
    DB_PASSWORD: "/prod/db/password"

  secrets-manager:                   # Secure secret rotation + encryption
    API_KEY: "prod/api/key"
```

| Block           | Meaning                                     |
| --------------- | ------------------------------------------- |
| variables       | Hard-coded ENV for build                    |
| parameter-store | Secure values (SSM Parameter Store)         |
| secrets-manager | Sensitive tokens/keys (AWS Secrets Manager) |

Access inside build:

```bash
echo $STAGE
echo $DB_PASSWORD        # Pulled securely at runtime
```

---

## 🔥 3) phases — THE HEART OF BUILDSPEC

Execution order:

```
install → pre_build → build → post_build
```

### 📌 install

Where dependencies are installed.

```yaml
install:
  runtime-versions:
    nodejs: 18
    java: corretto17
  commands:
    - npm install
    - mvn clean install -DskipTests
```

`runtime-versions` accepts:

| lang   | example                     |
| ------ | --------------------------- |
| nodejs | 18, 20                      |
| java   | corretto11, corretto17      |
| python | 3.10, 3.11                  |
| docker | enabled via privileged mode |

---

### 📌 pre_build

Run before actual build — usually tests or auth.

```yaml
pre_build:
  commands:
    - npm test
    - aws ecr get-login-password ... | docker login
```

---

### 📌 build

MAIN BUILD executes here.

```yaml
build:
  commands:
    - npm run build
    - mvn package
    - python build.py
```

---

### 📌 post_build

Packaging + pushing artifacts + deployment triggers

```yaml
post_build:
  commands:
    - zip -r output.zip .
    - docker push $IMAGE
    - echo "Build Completed!"
```

---

## 🔥 4) artifacts — final output (important!)

```yaml
artifacts:
  files:
    - output.zip
    - build/**/*
  discard-paths: yes
```

Artifacts go → S3 or CodePipeline next stage.

Multiple artifacts supported:

```yaml
secondary-artifacts:
  reports:
    files: coverage.xml
  binaries:
    files: target/*.jar
```

---

## 🔥 5) cache — build faster

```yaml
cache:
  paths:
    - node_modules/**/*
    - /root/.m2/repository/**/*   # Maven dependencies
```

Useful when build repeats — saves 70% time.

---

## 🔥 6) reports — Test reporting

```yaml
reports:
  junit-tests:
    files:
      - reports/junit/*.xml
    file-format: JUNITXML
```

Report appears in CodeBuild UI.

---

# 🧠 Summary BuildSpec Quick Flash Card

```
version         = YAML schema version
env             = variables + secret store
phases          = install -> pre_build -> build -> post_build
artifacts       = build outputs (zip, jar, dist)
cache           = speeds up future builds
reports         = JUnit/Jest/PyTest outputs
```

Now you TRULY understand `buildspec.yml`.

---

# 🟩 PART–2 → `appspec.yml` (FULL BREAKDOWN)

This tells CodeDeploy exactly **how to deploy artifacts**.

Deployment types affect syntax:

| Platform    | appspec format                         |
| ----------- | -------------------------------------- |
| EC2/On-prem | **YAML 0.0**                           |
| ECS         | **YAML version 1**                     |
| Lambda      | **YAML version 0.0** (Resources block) |

We cover each.

---

# 🔥 EC2 + On-Prem AppSpec (MOST ASKED)

```yaml
version: 0.0                # only for EC2
os: linux

files:
  - source: /build          # from artifact
    destination: /var/www/app
    overwrite: true

hooks:
  BeforeInstall:
    - location: scripts/stop_server.sh
      timeout: 300

  AfterInstall:
    - location: scripts/install_dependencies.sh

  ApplicationStart:
    - location: scripts/start_server.sh
      runas: ec2-user

  ValidateService:
    - location: scripts/health_check.sh
      timeout: 60
```

---

## 🔥 EC2 Lifecycle Hooks Explained

| Hook                 | Purpose                               |
| -------------------- | ------------------------------------- |
| **BeforeInstall**    | Stop services / backup files          |
| **AfterInstall**     | Copy new build / install deps         |
| **ApplicationStart** | Start service (pm2/systemctl)         |
| **ValidateService**  | HEALTH-CHECK — rollback triggers here |

This is where 80% deployments fail — so interviewers LOVE this topic.

---

### Example Scripts

stop_server.sh

```bash
sudo systemctl stop nginx || true
```

install_dependencies.sh

```bash
pip install -r requirements.txt
npm install
```

start_server.sh

```bash
sudo systemctl restart nginx
```

validate.sh

```bash
curl -f http://localhost/ || exit 1
```

If validate fails → CodeDeploy **ROLLS BACK** automatically.

---

# 🔥 ECS (Docker) AppSpec

```yaml
version: 1
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "myapp:42"
        LoadBalancerInfo:
          ContainerName: "app"
          ContainerPort: 80
```

Used for containerized deployments.

Traffic Shifting via CodeDeploy:

```
Blue Task → Running
Green Task → New Deployment
↓
Traffic Shift → Monitor → Rollback If Failed
```

---

# 🔥 Lambda AppSpec

```yaml
version: 0.0
Resources:
  - myFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: "MyLambda"
        Alias: "LIVE"
        CurrentVersion: 3
        TargetVersion: 4
```

Supports gradual rollout:

| Strategy | Meaning |
|---|
| Canary10Percent5Minutes | 10% → wait → 100% |
| Linear10PercentEvery1Minute | Gradual |
| AllAtOnce | Instant shift |

---

# 🧠 AppSpec Summary Flash Card

```
version: 0.0         # EC2 / Lambda
version: 1           # ECS
files                 # copy from artifact to EC2 path
hooks                 # lifecycle deployment scripts
ValidateService       # decides rollback
```

You now understand appspec **deep like architecture engineers.**

---

# 🎯 Interview-Grade Questions

| Question                    | Expected Answer                                                   |
| --------------------------- | ----------------------------------------------------------------- |
| Role of buildspec?          | Defines how CodeBuild builds code, tests & packages outputs       |
| Role of appspec?            | Defines deployment workflow — scripts & lifecycle hooks           |
| When ValidateService fails? | Automatic rollback triggered                                      |
| Name EC2 hooks in order     | BeforeInstall → AfterInstall → ApplicationStart → ValidateService |
| Blue/Green supported?       | Yes — through CodeDeploy for EC2/ECS/Lambda                       |
| Deployment failure cause?   | Wrong hooks, missing script perms, validate health fail           |

If you say this — **you sound senior.** 💥

---


What do you choose? 🚀
