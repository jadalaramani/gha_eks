# gha_eks
Below is **the complete, end-to-end setup** to:

### ✅ Install Self-Hosted GitHub Runner

### ✅ Create EKS Cluster using GitHub Actions (without Terraform)

### ✅ Connect kubectl and verify cluster


# ⭐ **PHASE 1 — Setup Self-Hosted GitHub Runner (EC2)**

---

## **1️⃣ Launch an EC2 Instance**

Recommended OS: **Amazon Linux 2**
Size: **t3.medium or t3.large**
Disk: **20–30 GB**
Security Groups:
✔ Allow SSH from your IP
✔ Allow outbound HTTPS (443)

---

## **2️⃣ Create a new user for runner**

```bash
sudo useradd -m runner
sudo passwd runner
sudo usermod -aG wheel runner
```
---

# **🔹 Step 2: Switch to runner user and go inside the folder**

```bash
su - runner
```
---

## **4️⃣ Download GitHub Actions Runner**

Create folder:

```bash
mkdir actions-runner && cd actions-runner
```

Download:

```bash
curl -o actions-runner-linux-x64-2.329.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.329.0/actions-runner-linux-x64-2.329.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.329.0.tar.gz
```

---

## **5️⃣ Install .NET Core dependencies**

```bash
sudo yum install -y libicu
sudo yum install -y icu
sudo yum install -y icu-libs
sudo yum install -y curl jq tar gzip
sudo ./bin/installdependencies.sh
```

(Required for GitHub runner to work)

---

## **6️⃣ Configure the Runner**

Get token here:
**GitHub → Repo → Settings → Actions → Runners → New runner**

Then run:

```bash
./config.sh --url https://github.com/jadalaramani/gha_eks --token <TOKEN>
```

---

## **7️⃣ Start the runner**

```bash
./run.sh
```

You should see:

```
√ Connected to GitHub
Listening for Jobs...
```

---

# ⭐ **PHASE 2 — Install Tools on Runner (AWS CLI, kubectl, eksctl)**

---

## **8️⃣ Install AWS CLI**

```bash
sudo yum install -y awscli
```

---

## **9️⃣ Install kubectl**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

---

## **🔟 Install eksctl**

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
| tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
```

---

## **1️⃣1️⃣ Configure AWS Credentials**

Create an IAM user/policy with admin or EKS permissions.

On runner:

```bash
aws configure
```

Enter:

* Access Key
* Secret Key
* Region 

---

# ⭐ **PHASE 3 — GitHub Actions Workflow (Create EKS)**

Create this file in your repo:

📌 **.github/workflows/create-eks.yml**

```yaml
name: Create EKS Cluster

on:
  workflow_dispatch:

jobs:
  create-eks:
    runs-on: self-hosted

    steps:
      - name: Checkout Repo
        uses: actions/checkout@v4

      - name: Create EKS Cluster
        run: |
          eksctl create cluster \
            --name demo-eks \
            --region ap-south-1 \
            --node-type t3.medium \
            --nodes 2 \
            --nodes-min 2 \
            --nodes-max 4

      - name: Update Kubeconfig
        run: |
          aws eks update-kubeconfig --name demo-eks --region us-east-1

  #    - name: Delete EKS Cluster
 #       run: |
  #        eksctl delete cluster \
   #         --name dev-eks \
    #        --region us-east-1

      - name: Test Cluster
        run: |
          kubectl get nodes
          kubectl get pods -A
          kubectl get svc -A
```

---

# ⭐ **PHASE 4 — Run the Pipeline**

Go to:

**GitHub → Actions → Create EKS Cluster → Run workflow**

Your EC2 runner will:

1. Run eksctl
2. Create EKS control plane
3. Add worker nodes
4. Configure kubeconfig
5. Test pods, nodes, services

---

# ⭐ **PHASE 5 — Verify Cluster (Manual check)**

SSH into EC2 runner:

```bash
aws eks update-kubeconfig --name demo-eks --region ap-south-1
```

Check everything:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get svc -A
kubectl cluster-info
```

---

 **DONE! Your EKS Cluster is created from a self-hosted GitHub Runner.**


or
👉 **“Add helm deployments”**
