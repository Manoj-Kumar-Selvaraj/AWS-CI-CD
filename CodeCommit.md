---

# 🔥 1.1 CodeCommit — Deep Explanation

### 📌 What is CodeCommit?

```
A secure, private Git-based source control service managed by AWS.
```

Just like **GitHub**, **GitLab**, **Bitbucket**, but:

* No external accounts (IAM based)
* Private by default
* Better for enterprise + regulated environments

---

### 🔥 Use Cases

| When to choose CodeCommit?  | Why                                                |
| --------------------------- | -------------------------------------------------- |
| Private enterprise repos    | IAM access > GitHub user access model              |
| No internet access allowed  | Works inside VPC with VPC endpoints                |
| Native AWS DevOps workflows | Integrates directly with CodeBuild/Deploy/Pipeline |

---

### 📁 Repository Structure

```
myrepo/
├── backend/
├── ui/
├── infra/
└── scripts/
```

Supports:

✔ Branching
✔ Pull Requests
✔ Merge approvals
✔ Webhooks

---

### 🟢 Create Repository

Console → CodeCommit → Create Repo

or CLI:

```bash
aws codecommit create-repository --repository-name devops-repo
```

---

### 🟢 Clone Repo (HTTPS)

```bash
git clone https://git-codecommit.<region>.amazonaws.com/v1/repos/devops-repo
```

Authenticate via AWS CLI → avoids username/password.

---

### 🟢 Clone Repo (SSH)

```bash
git clone ssh://git-codecommit.<region>.amazonaws.com/v1/repos/devops-repo
```

You must upload public SSH key in IAM.

---

### 🟢 IAM Access Model

Policies to allow access:

```json
{
  "Effect": "Allow",
  "Action": "codecommit:*",
  "Resource": "*"
}
```

Fine-grained example:

```json
"codecommit:GitPush",
"codecommit:GitPull",
"codecommit:MergePullRequestByFastForward"
```

You can restrict by:

✔ Branch
✔ Tag
✔ IP
✔ MFA required

---

### 📌 CodeCommit rarely stands alone — always used with CI/CD

Typical flow:

```
Developer → Commit → CodeCommit
                      ↓
                 CodePipeline Trigger
                      ↓
                 CodeBuild → Build Artifact
                      ↓
                 CodeDeploy → Deploy to Env
```

---
