### 🔥 **1.3 AWS CodeDeploy — Deep, Detailed Breakdown**

CodeDeploy is the **deployment engine** in AWS CI/CD.
If CodeBuild = *compile + build*, then CodeDeploy = *release + deploy*.

You will understand it **inside-out**, including:

```
✔ Deployment Types (In-Place vs Blue/Green)
✔ AppSpec.yml — Full Syntax Line-by-Line
✔ Lifecycle Hooks (BeforeInstall, AfterInstall, ValidateService etc.)
✔ EC2 / ECS / Lambda Deployment Patterns
✔ Rollback Strategy + Traffic Shifting
✔ CI/CD Pipeline Integration (Real flow)
✔ Interview-grade explanations
```

---

# 📌 What is CodeDeploy?

```
A fully-managed deployment service that automates releasing applications
to EC2, ECS, Lambda, and even On-Prem servers.
```

Equivalent in traditional world:

| Jenkins                       | AWS Equivalent               |
| ----------------------------- | ---------------------------- |
| Deploy Stage                  | CodeDeploy                   |
| Shell copy to servers         | Hooks + AppSpec              |
| Rolling/Blue-Green Deployment | CodeDeploy Deployment Groups |

---

# 🟨 Deployment Modes

| Mode                      | How It Works                                     | Use Case                  |
| ------------------------- | ------------------------------------------------ | ------------------------- |
| **In-Place Deployment**   | Update existing EC2 instance directly            | Small or internal apps    |
| **Blue/Green Deployment** | Create new environment & shift traffic gradually | Production-grade releases |

Diagram:

```
In-Place Deployment     Blue/Green Deployment
───────────────         ──────────────────────────
EC2 Updated Directly    Blue (Old) ← 100% traffic
 ⤷ downtime possible     ↓
                         Green (New) build
                          ↓
                       Traffic Shift → 100%
```

Blue/Green = Zero-Downtime + Safe Rollback.

---

# 🔥 Core File — `appspec.yml` (Most Important)

This tells CodeDeploy **how to deploy artifact & which scripts to run**.

EC2/LINUX full syntax:

```yaml
version: 0.0
os: linux

files:
  - source: /                    # from artifact
    destination: /var/www/app    # deploy target path
    overwrite: true

hooks:
  BeforeInstall:
    - location: scripts/stop.sh
      timeout: 300
  AfterInstall:
    - location: scripts/install-dependencies.sh
  ApplicationStart:
    - location: scripts/start.sh
      runas: ec2-user
  ValidateService:
    - location: scripts/validate.sh
      timeout: 60
```

---

## 🧩 Understanding Every Block

### `version`

Always `0.0` for EC2.

### `os`

`linux` or `windows`.

---

### `files`

| Key         | Purpose                             |
| ----------- | ----------------------------------- |
| source      | location inside build artifact      |
| destination | target directory inside EC2 instace |
| overwrite   | true → overwrite existing files     |

Example:

```
source: /       → artifact root
destination: /var/www/html
```

---

### `hooks` — The Heart of CodeDeploy

Hooks run scripts in order.
This is where deployments succeed or break.

| Hook Name            | Meaning              | Typical Script               |
| -------------------- | -------------------- | ---------------------------- |
| **BeforeInstall**    | Stop old app, backup | stop server, clean directory |
| **AfterInstall**     | Extract new files    | install dependencies         |
| **ApplicationStart** | Start application    | systemctl restart, pm2 start |
| **ValidateService**  | Final check          | curl health-check endpoint   |

Execution Order:

```
BeforeInstall → AfterInstall → ApplicationStart → ValidateService
```

If `ValidateService` fails → rollback.

---

# 📌 Example Scripts

stop.sh:

```bash
#!/bin/bash
systemctl stop myapp || true
```

install-dependencies.sh:

```bash
pip install -r requirements.txt
npm install
```

start.sh:

```bash
systemctl start myapp
```

validate.sh:

```bash
curl -f http://localhost:8080/health || exit 1
```

---

# 🔥 Lambda Deployment Using CodeDeploy

Simple versioning + traffic shifting:

```yaml
version: 0.0
Resources:
  - myFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: "myapp"
        Alias: "live"
        CurrentVersion: 3
        TargetVersion: 4
```

Traffic shifting strategy:

| Deployment Config             | Meaning                   |
| ----------------------------- | ------------------------- |
| `Canary10Percent5Minutes`     | 10% traffic → wait → 100% |
| `Linear10PercentEvery1Minute` | 10% increase each minute  |
| `AllAtOnce`                   | Immediate switch          |

---

# 🔥 ECS Deployment via CodeDeploy

Used for containerized apps:

```
Task Definition Old (Blue) ← Running
Task Definition New (Green) ← Created
↓
Traffic shifting via ALB target groups
↓
If fail → rollback to Blue
```

AppSpec for ECS looks different:

```yaml
version: 1
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: new-task:1
        LoadBalancerInfo:
          ContainerName: "web"
          ContainerPort: 80
```

---

# 🧠 How CodeDeploy Fits CI/CD Pipeline

```
CodeBuild → produces artifact
 ↓
Store in S3 / ECR
 ↓
CodePipeline triggers CodeDeploy
 ↓
Deploy to EC2 / ECS / Lambda
 ↓
Validate & Shift Traffic
```

Full AWS CI/CD flow:

```
Source → Build → Test → Deploy → Validate → Prod
```

---

# 🔥 Real Interview Questions (Answer Format Included)

| Question                                   | Perfect Short Answer                                                       |
| ------------------------------------------ | -------------------------------------------------------------------------- |
| What is CodeDeploy?                        | A managed deployment service for EC2, ECS, Lambda & OnPrem                 |
| What file is required for deployment?      | `appspec.yml`                                                              |
| What are lifecycle hooks?                  | Scripts run during deploy: BeforeInstall → AfterInstall → Start → Validate |
| Difference between In-Place vs Blue/Green? | In-place replaces existing env; Blue/Green creates new one + traffic shift |
| How does rollback work?                    | ValidateService failure → revert to last known good                        |
| Can CodeDeploy work without CodePipeline?  | Yes. CLI + Manually + API + GitHub Webhooks                                |

If you can answer like this — you are **deploy-ready.**

---
