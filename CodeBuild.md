CodeBuild is the **build powerhouse** — the AWS equivalent of Jenkins build nodes.


You will use it to:

* Compile Java (Maven/Gradle)
* Run Python/Node builds
* Run tests (JUnit/Jest/Pytest)
* Generate artifacts (JAR, .zip, dist folder, docker images)
* Push images to ECR
* Run SonarQube scans
* Perform security scanning (Trivy)

This is one of the **core skills** for AWS DevOps engineers.

We will also cover some advanced concpts:

✔ Environment types (standard vs custom images)
✔ Compute sizes, VPC builds, caching, artifacts storage
✔ S3 integration
✔ IAM roles (VERY IMPORTANT)
✔ Logs (CloudWatch + S3)
✔ Webhooks / Triggers
✔ Build Badges (Visibility)
✔ Scaling behavior
✔ How CodeBuild fits into CI/CD architectures
✔ Advanced patterns like parallel builds & test matrix
✔ Cost optimization
✔ Common interview real questions


---

# 📌 What Exactly is CodeBuild?

```
Managed build service that runs your code, installs dependencies,
builds artifacts, runs tests, and outputs deployable packages.
```

No VM maintenance, autoscaling, pay-per-minute.

Equivalent to:

| Traditional             | AWS Equivalent          |
| ----------------------- | ----------------------- |
| Jenkins Agent/Slave     | CodeBuild               |
| Jenkinsfile build stage | buildspec.yml           |
| Build Node Machine      | Build Environment Image |

---

# 🏗 CodeBuild Core Components

| Component | Purpose |
|---|
| **Environment** | Runtime: Java, Node, Python, Docker |
| **Compute Type** | build.power.small / medium / large |
| **Source Provider** | CodeCommit / GitHub / Bitbucket / S3 |
| **Artifacts** | Output files to S3 or CodePipeline |
| **buildspec.yml** | Defines commands, phases, artifacts (VERY IMPORTANT) |

---

# 🔥 The Heart of CodeBuild = `buildspec.yml`

This file controls everything that CodeBuild does.

Full Structure:

```yaml
version: 0.2

env:
  variables:
    STAGE: "dev"
  parameter-store:
    DB_PASSWORD: "/prod/db/pass"
  secrets-manager:
    API_KEY: "prod/api/key"

phases:
  install:
    runtime-versions:
      java: corretto17
      nodejs: 18
      python: 3.10
    commands:
      - echo "Installing dependencies"
      - npm install
  pre_build:
    commands:
      - echo "Running unit tests..."
      - npm test
  build:
    commands:
      - echo "Building application"
      - npm run build
  post_build:
    commands:
      - echo "Packaging artifact"
      - zip -r output.zip build/

artifacts:
  files:
    - output.zip
    - build/**/*
  discard-paths: yes

cache:
  paths:
    - node_modules/**/*
```

Breakdown of **EVERY KEYWORD**:

| Keyword             | Meaning                                         |
| ------------------- | ----------------------------------------------- |
| `env`               | Set environment vars, secrets, Parameters Store |
| `phases.install`    | Install dependencies like npm/maven             |
| `phases.pre_build`  | Optional test phase                             |
| `phases.build`      | Main build/compile step                         |
| `phases.post_build` | Packaging, pushing images, notifications        |
| `artifacts`         | Final build output to CodePipeline/S3           |
| `cache`             | Speed up builds (stores dependencies)           |

🚨 Memorize this — interviewer will ask.

---

# 🧪 Examples By Language (Must Know)

### Maven / Java Build

```yaml
phases:
  install:
    commands:
      - mvn clean install -DskipTests
artifacts:
  files:
    - target/*.jar
```

---

### Node/React Build

```yaml
phases:
  install:
    commands: npm install
  build:
    commands: npm run build
artifacts:
  files: build/**/*
```

---

### Python Build

```yaml
phases:
  install:
    commands: pip install -r requirements.txt
  build:
    commands: zip -r app.zip .
artifacts:
  files:
    - app.zip
```

---

# 🐳 Docker Build + Push to ECR (MOST IMPORTANT)

```yaml
phases:
  pre_build:
    commands:
      - aws ecr get-login-password --region $AWS_REGION |
        docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
      - IMAGE="$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/myapp:$CODEBUILD_RESOLVED_SOURCE_VERSION"
  build:
    commands:
      - docker build -t $IMAGE .
      - docker push $IMAGE
artifacts:
  files: []
```

This alone makes you "AWS CI/CD Ready for Kubernetes ECS/EKS."

---

# 🔥 CodeBuild in CodePipeline

Very common stage block:

```yaml
BuildStage:
  Name: Build
  Actions:
    - Name: CodeBuildAction
      ActionTypeId:
        Category: Build
        Owner: AWS
        Provider: CodeBuild
        Version: "1"
      InputArtifacts:
        - Name: SourceOutput
      OutputArtifacts:
        - Name: BuildOutput
```

---

# 🧠 Interview Pointers

> **What is CodeBuild?**
> → A serverless build service replacing Jenkins agents.

> **What file controls build steps?**
> → `buildspec.yml`

> **How do you pass secure values like passwords?**
> → SSM Parameter Store / Secrets Manager in `env` block.

> **How to trigger CodeBuild?**
> → CodePipeline, EventBridge, Git webhook, Scheduled Cron.

---


Perfect — now we take **AWS CodeBuild deeper BEYOND buildspec.yml**.

We will cover everything that makes CodeBuild production-grade in CI/CD:

```
✔ Environment types (standard vs custom images)
✔ Compute sizes, VPC builds, caching, artifacts storage
✔ S3 integration
✔ IAM roles (VERY IMPORTANT)
✔ Logs (CloudWatch + S3)
✔ Webhooks / Triggers
✔ Build Badges (Visibility)
✔ Scaling behavior
✔ How CodeBuild fits into CI/CD architectures
✔ Advanced patterns like parallel builds & test matrix
✔ Cost optimization
✔ Common interview real questions
```

Let’s break it down one-by-one.

---

# 🔥 1) CodeBuild Build Environments

There are **3 ways to run CodeBuild**:

| Type                        | Meaning                                            | Use case                        |
| --------------------------- | -------------------------------------------------- | ------------------------------- |
| **Managed Image**           | AWS provides Node/Python/Java/Docker pre-installed | Best for standard builds        |
| **Custom Image (from ECR)** | Your own Docker image w/ dependencies              | When builds need specific tools |
| **Privileged Mode**         | Enables Docker inside CodeBuild                    | Needed for Docker build + push  |

Example Build Env:

```
AWS Managed: aws/codebuild/standard:7.0
```

Custom Image:

```
ECR Repo → my-custom-build-image:latest
```

---

# 🔥 2) Compute Types (Which machine you get)

| computeType             | vCPU   | RAM       |
| ----------------------- | ------ | --------- |
| `BUILD_GENERAL1_SMALL`  | 3 vCPU | 7 GB RAM  |
| `BUILD_GENERAL1_MEDIUM` | 4 vCPU | 16 GB RAM |
| `BUILD_GENERAL1_LARGE`  | 8 vCPU | 34 GB RAM |

Rule of thumb:

| Project type        | Recommended |
| ------------------- | ----------- |
| Node/React/Angular  | SMALL       |
| Java (Spring Boot)  | MEDIUM      |
| Docker image builds | LARGE       |

---

# 🔥 3) Artifacts Handling

After build, output goes to:

✔ S3
✔ CodePipeline
✔ ECR (if docker)
✔ Report Groups (test reports)

Options:

```yaml
artifacts:
  type: zip
  files:
    - build/**
  discard-paths: yes
```

Multiple artifacts:

```yaml
secondary-artifacts:
  reports:
    files:
      - coverage.xml
  binaries:
    files:
      - target/*.jar
```

---

# 🔥 4) Caching (Build Speed Boost) — VERY IMPORTANT

Enables 3–5x faster npm/maven builds.

```yaml
cache:
  paths:
    - '/root/.m2/**/*'             # Maven
    - 'node_modules/**/*'          # Node
```

AWS cache storage = S3 backend.

Without cache:

```
npm install = slow
mvn dependency:resolve = slow
```

With cache:

```
Reuses previous dependencies → fast
```

---

# 🔥 5) IAM Role (MOST CRITICAL INTERVIEW TOPIC)

CodeBuild uses **Execution Role**, NOT user's IAM.

Role requires permissions:

| Action          | Service Permission                            |
| --------------- | --------------------------------------------- |
| Pull code       | CodeCommit/GitHub Read                        |
| Save artifacts  | S3 PutObject                                  |
| Push to ECR     | ecr:PutImage, ecr:BatchCheckLayerAvailability |
| Read parameters | SSM GetParameter                              |
| Write logs      | CloudWatchLogs                                |

Example IAM policy snippet:

```json
{
  "Effect": "Allow",
  "Action": [
    "ecr:*",
    "logs:*",
    "s3:*",
    "ssm:GetParameter",
    "secretsmanager:GetSecretValue"
  ],
  "Resource": "*"
}
```

🔥 If IAM is wrong → CodeBuild FAILS even if buildspec is correct.

---

# 🔥 6) Running CodeBuild inside VPC (for private services)

```
Subnets → Private
SecurityGroup → Controls access
No public internet unless NAT exists
```

Used when:

✔ App dependencies are internal
✔ Building for RDS/internal service
✔ CodeConnect to VPC resources

Diagram:

```
CodeBuild → VPC ENI → Private Subnet → Database → Internal API
```

---

# 🔥 7) Logs

| Output          | Purpose                 |
| --------------- | ----------------------- |
| CloudWatch Logs | Console output          |
| S3 Logs         | Permanent/store reports |

Enable CW logs in Project Settings.

---

# 🔥 8) Metrics & Webhooks

You can trigger builds:

* CodePipeline (most common)
* Git push (webhooks)
* CloudWatch Events
* Manual run

Webhook example:

```
Push → Build auto triggers (like GitHub Actions)
```

Build Badge:

```
[![Build Status](https://codebuild.XYZ)]()
```

Add in README like GitHub Actions badge.

---

# 🔥 9) Scaling / Parallel Builds

CodeBuild is **serverless**, so:

```
No queue
No concurrency limit
No auto-scaling config needed
You pay per-minute only
```

Multiple builds can run simultaneously.

You can also run **test matrix** with multiple compute configs.

---

# 🔥 10) Advanced Build Patterns

| Task                        | Method                                    |
| --------------------------- | ----------------------------------------- |
| SonarQube Scan              | buildspec.yml → mvn sonar:sonar           |
| Multi-env build dev/qa/prod | Parameterized buildspec                   |
| Multi-artifact packaging    | secondary-artifacts                       |
| Multi-language monorepo     | separate CodeBuild projects               |
| Canary releases             | integrate with CodeDeploy or Lambda alias |

Example: Multi-env build

```yaml
env:
  variables:
    ENV: $ENVIRONMENT
phases:
  build:
    commands:
      - if [ "$ENV" == "prod" ]; then npm run build:prod; else npm run build:dev; fi
```

---

# 🔥 CodeBuild Architecture Placement

Most common architecture:

```
CodeCommit/GitHub → CodePipeline → CodeBuild → S3 Or ECR → CodeDeploy → EC2/ECS/Lambda
```

Flow:

```
Source → Build → Artifact → Deploy
```

---

# 🧠 CodeBuild Interview Questions

| Question                         | Best Answer Summary                                  |
| -------------------------------- | ---------------------------------------------------- |
| What is CodeBuild?               | Serverless CI engine that compiles/tests/builds code |
| File used to define build steps? | `buildspec.yml`                                      |
| How are secrets passed?          | SSM Parameter Store / Secrets Manager                |
| Can CodeBuild build Docker?      | Yes with privileged mode                             |
| How do you make builds faster?   | Cache npm/maven + bigger compute for heavy builds    |
| How do you run inside VPC?       | Assign Subnet + SG + VPC config in project           |
| Difference vs CodePipeline?      | CodePipeline orchestrates, CodeBuild executes        |

If you answer like this — you sound **mid-senior level**.

---

### 🔥 **CodeBuild deep = COMPLETE.**

Next Subtopic:

```
1.3 CodeDeploy (We go deep with appspec.yml + Blue/Green Deployments)
```

Reply:

```
Next (1.3 CodeDeploy)
```

We will break AppSpec.yml line-by-line like we just did with CodeBuild.


We will now study **Blue/Green Deployments**, Hooks (BeforeInstall, AfterInstall), Lambda + ECS deploys, Rollbacks — DEEP.
