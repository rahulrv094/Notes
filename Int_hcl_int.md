# ✅ **SRE / DevOps Interview Q&A (Full Set)**

*(All questions from your list covered clearly.)*

---

# 🔥 **ROUND 1 – Q&A**

---

### **1. Q: Repo “abcd” has 4 developers + 2 SREs. As an SRE, how do you deploy to production?**

**A:**

1. Developers create PR → merged into `main`.
2. CI pipeline runs: build, test, security scan.
3. Artifact pushed to registry (ECR/JFrog).
4. CD pipeline deploys to Dev → QA → Staging.
5. SRE validates health, logs, metrics.
6. After approval, SRE deploys to Production using Helm/Jenkins/ArgoCD.
7. Post-deployment monitoring + rollback strategy.

---

### **2. Q: Explain the CI/CD flow for the abcd repository.**

**A:**
**CI Stages:**

* Checkout code
* Linting & static code analysis
* Unit tests
* Build artifact
* Security scan
* Push to registry

**CD Stages:**

* Deploy to Dev
* Integration tests
* Approvals
* Deploy to QA → Staging → Production

---

### **3. Q: Explain your branching strategy & promotion to environments.**

**A:**
I use **Gitflow**:

* `feature/*` → Development work
* `develop` → Dev environment
* `release/*` → QA/Staging
* `main` → Production
* Tags → Production releases

Promotion happens through PR merges into respective branches.

---

### **4. Q: Write your name 10 times in any script.**

**A (Bash):**

```bash
for i in {1..10}; do echo "Rahul"; done
```

**A (Python):**

```python
for i in range(10):
    print("Rahul")
```

---

### **5. Q: Difference between Terraform and Ansible?**

**A:**

| Terraform                   | Ansible                   |
| --------------------------- | ------------------------- |
| Infra provisioning          | Config management         |
| Declarative                 | Procedural                |
| Maintains state             | No state                  |
| Immutable infra             | Mutable changes           |
| Best for creating resources | Best for updating configs |

---

### **6. Q: If there is an issue in Linux, how will you check?**

**A:**

* `top`, `htop` → CPU/RAM
* `free -m` → Memory
* `df -h` → Disk
* `iostat`, `vmstat` → IO
* `journalctl -xe`, `/var/log/messages` → Logs
* `systemctl status <service>`
* `ss -lntp` → Ports

---

### **7. Q: How do you store username & password in Terraform?**

**A:**
Use:

* Variables marked `sensitive = true`
* `.tfvars` (not committed)
* Prefer AWS Secrets Manager / SSM Parameter Store

Example:

```hcl
variable "db_password" {
  type = string
  sensitive = true
}
```

---

### **8. Q: How to get data flow on a URL address?**

**A:**

* `curl -v https://url`
* `traceroute url`
* `dig url`
* Browser dev network logs
* Load balancer access logs

---

### **9. Q: How do you troubleshoot application down?**

**A:**

1. Check service: `systemctl status app`
2. Check logs
3. CPU/memory/disk
4. Check DB/Redis connectivity
5. Restart service
6. Check deployment version
7. Verify load balancer health checks

---

# 🔥 **ROUND 1 (Additional Questions)**

---

### **10. Q: How to check if something happened on a Linux server?**

**A:**

* `/var/log/messages`
* `journalctl -xe`
* `dmesg`
* `uptime`
* `sar`, `vmstat`, `iostat`

---

### **11. Q: Difference between Terraform & Ansible?**

**A:** *(Same as Q5 above)*

---

### **12. Q: Script to print your name multiple times?**

Covered above.

---

### **13. Q: How do you secure sensitive information in Terraform?**

**A:**

* Use AWS Secrets Manager or SSM
* `sensitive = true` variables
* Store values in Terraform Cloud variables
* Don’t commit tfvars to Git

---

### **14. Q: How to deploy a Java app using CI/CD?**

**A:**

1. Build WAR/JAR using Maven.
2. Dockerize the app.
3. Push image to ECR.
4. Deploy using Helm to EKS OR ECS service update.
5. Validate using health checks.

---

### **15. Q: How to segregate environments in pipelines?**

**A:**

* Different branches → Dev/QA/Prod
* Different workspace → Terraform
* Different namespaces → EKS
* Approval gates → Prod

---

### **16. Q: Do you have incident management experience?**

**A:**
Yes. Involves:

* Alert triage
* Logging analysis
* Running root-cause analysis
* Communicating updates
* Creating postmortems
* Preventing recurrence

---

### **17. Q: Explain end-to-end Terraform execution.**

**A:**

1. Write modules
2. Initialize → `terraform init`
3. Validate → `terraform validate`
4. Plan → `terraform plan`
5. Apply → `terraform apply`
6. Maintain state (local/remote)
7. Use workspaces for environments

---

# 🔥 **ROUND 2 – Q&A**

---

### **1. Q: Which monitoring tools have you worked with?**

**A:**
Grafana, Prometheus, CloudWatch, ELK, Dynatrace, Datadog.
Used for:

* Metrics
* Logs
* Alerting
* Dashboards
* SLO/SLA monitoring

---

### **2. Q: How to design infra with HA, security, reliability?**

**A:**

* Multi-AZ architecture
* Auto-scaling
* Private subnets
* WAF + SGs + NACLs
* IAM least privilege
* Backup policies
* Monitoring + Alerts
* DR strategy with RTO/RPO

---

### **3. Q: When will you choose Terraform vs Ansible?**

**A:**

* Terraform → create infra
* Ansible → configure apps
  Use both together often.

---

### **4. Q: Terraform code to get OS of a server?**

**A:**

```hcl
data "aws_ami" "example" {
  most_recent = true
  owners      = ["amazon"]
}
output "os" {
  value = data.aws_ami.example.name
}
```

---

### **5. Q: Integrate Jenkins with GitHub/GitLab?**

**A:**

* Add GitHub webhook
* Use Git plugin / GitHub app
* Configure Jenkinsfile
* Add credentials in Jenkins
* Pipeline triggers on push/PR

---

### **6. Q: How are logs exported to Grafana?**

**A:**
Grafana Loki + Promtail
or
Grafana connected to ELK/CloudWatch/Datadog

---

### **7. Q: Other Terraform providers you used?**

**A:**
AWS, Azure, Helm, Kubernetes, GitHub, Cloudflare, Vault, Datadog.

---

# 🔥 **AWS / EKS QUESTIONS – Q&A**

---

### **1. Q: You worked more on Azure; what do you use in AWS for similar?**

**A:**

* Azure VM → AWS EC2
* Azure AKS → AWS EKS
* Azure SQL → RDS
* Azure Blob → S3
* Azure Monitor → CloudWatch

---

### **2. Q: Managed services to maintain cluster in AWS?**

**A:**

* EKS Managed Node Groups
* EKS Control Plane
* Fargate
* AWS AppMesh
* ECR

---

### **3. Q: Worked on ECR?**

**A:**
Yes:

* Create repo
* Authenticate (login)
* Push/pull images
* Lifecycle rules

---

### **4. Q: What is AWS Fargate?**

**A:**
Serverless compute for ECS/EKS.
No EC2 needed.

---

### **5. Q: If you need to create AWS infra using Python with state, what do you use?**

**A:**
**AWS CDK (Python)** → IaC
State maintained in CloudFormation.

---

### **6. Q: How do you deploy WAR file to AWS?**

**A:**
Options:

* Tomcat running on EC2
* Docker container running on ECS/EKS
* Elastic Beanstalk

---

### **7. Q: How to build a pipeline for .NET/Python app?**

**A:**

1. Code checkout
2. Run build/test
3. Package artifact
4. Terraform apply infra
5. Deploy app to EC2/EKS/ECS
6. Notify success

---
Here is your **ultra-compact, last-minute SRE/DevOps Interview Cheat Sheet** — perfect for quick revision before any round.

---

# ✅ **SRE / DevOps – Quick Revision CHEAT SHEET (Super Short)**

---

# 🔥 **1. CI/CD Flow (Most asked)**

**CI**
✔ Checkout → Lint → Test → Build → Security Scan → Push artifact (ECR/JFrog)

**CD**
✔ Deploy Dev → QA → Staging → Prod
✔ Manual approval → Helm/ArgoCD/Jenkins deploy
✔ Post-deploy verification (logs + metrics)

---

# 🔥 **2. Git Branching & Environment Promotion**

**Gitflow**

* `feature/*` → Dev
* `develop` → Dev environment
* `release/*` → QA/Staging
* `main` → Prod
* Tags → Production releases

---

# 🔥 **3. Terraform vs Ansible (10-sec answer)**

**Terraform** → Infra provisioning, declarative, state, immutable.
**Ansible** → Config mgmt, procedural, no state, mutable.

Use Terraform to **create infra**; Ansible to **configure inside it**.

---

# 🔥 **4. Linux Troubleshooting Commands**

| Area    | Commands                                                 |
| ------- | -------------------------------------------------------- |
| CPU/Mem | `top`, `htop`, `free -m`, `vmstat`                       |
| Disk    | `df -h`, `du -sh *`                                      |
| Network | `ss -lntp`, `netstat`, `curl`, `ping`, `traceroute`      |
| Logs    | `journalctl -xe`, `/var/log/messages`, `/var/log/syslog` |
| IO      | `iostat`, `sar`                                          |

---

# 🔥 **5. App Down? Immediate Troubleshooting Steps**

1. `systemctl status <service>`
2. Check logs
3. CPU/memory/disk
4. Check port (ss/netstat)
5. Check DB connectivity
6. Restart service
7. Check LB health checks
8. Rollback if needed

---

# 🔥 **6. Secure Sensitive Data in Terraform**

✔ `sensitive = true`
✔ Store values in tfvars (not in Git)
✔ Use **AWS Secrets Manager / SSM Parameter Store**
✔ Terraform Cloud workspace variables

---

# 🔥 **7. Java/Python/.NET App Deployment on AWS**

**Flow:**

1. Build → Test → Package
2. Dockerize
3. Push image → ECR
4. Deploy via Helm to EKS OR ECS service update
5. Health checks + auto-scaling

---

# 🔥 **8. AWS High Availability Checklist**

* Multi-AZ setup
* Auto Scaling Groups
* Private subnets
* NAT gateway
* Load Balancers
* RDS Multi-AZ
* CloudWatch alerts
* Backups + DR plan

---

# 🔥 **9. Monitoring Concepts**

**Metrics** → Numbers (CPU, Latency, Memory)
**Logs** → Events (App logs, system logs)
**Traces** → Request flow (distributed systems)

Tools: Prometheus, Grafana, CloudWatch, ELK, Loki.

---

# 🔥 **10. Network Troubleshooting (Linux)**

**Commands:**
`ping`, `curl -v`, `traceroute`, `dig`, `ss -lntp`, `tcpdump -i eth0`
**Flow:**
DNS → Routing → Firewall → App → DB.

---

# 🔥 **11. Terraform Basics**

✔ `terraform init` – download providers
✔ `terraform plan` – preview
✔ `terraform apply` – execute
✔ `terraform destroy` – delete
✔ `terraform state list` – list resources
✔ Workspaces → multi-env
✔ Remote state → S3 + DynamoDB lock

---

# 🔥 **12. AWS EKS Quick Notes**

* Control plane managed by AWS
* Worker nodes → EC2 or Fargate
* Use IAM roles for service accounts (IRSA)
* CI/CD deployment → Helm + ArgoCD

---

# 🔥 **13. Incident Management (Short answer)**

* Acknowledge alert
* Assess impact
* Check logs/metrics
* Fix/rollback
* RCA report
* Prevent recurrence

---

# 🔥 **14. URL/Data Flow Troubleshooting**

* DNS → `dig`, `nslookup`
* TCP → `curl`, `ss -lntp`
* Network → `traceroute`, `ping`
* Application → Logs, LB health checks

---

# 🔥 **15. Jenkins Integration Quick Notes**

* Connect via webhook
* Use Jenkinsfile
* Store credentials in Jenkins credentials store
* Pipeline stages: build → scan → deploy
----

Here are **quick, crisp interview-ready answers** for all your questions:

---

# ✅ **Quick Answers – SRE / DevOps / AWS / Terraform / K8s**

---

## **1. How will you host a 3-tier application on AWS using EKS with high availability if an entire region goes down?**

**Answer (Quick):**

* Use **multi-region EKS clusters** (active-active or active-passive).
* Deploy each tier (frontend, backend, DB) across **multiple AZs**.
* Use **Aurora Global Database** or **DynamoDB Global Tables** for DB HA.
* Use **Route53 latency-based routing** or **failover routing** for region failover.
* Store Docker images in **ECR (replicated)**.
* Use **CI/CD (ArgoCD/GitOps)** to sync both regions.
* Use **global load balancer** – CloudFront + ALB.
* Enable **cross-region S3 replication**, backup, and state replication.

---

## **2. What is a CNI?**

**Quick answer:**
**Container Network Interface (CNI)** is a plugin specification that defines how Pods get IP addresses and network connectivity inside Kubernetes.

---

## **3. Advantages & Disadvantages of CNI**

**Advantages:**

* Pod-level IP assignment
* High performance networking
* Integration with cloud networks (AWS VPC CNI)
* Scalable & pluggable (Calico, Cilium, Weave)

**Disadvantages:**

* Complex to troubleshoot
* Different CNIs behave differently (policy, routing)
* Can become bottleneck if misconfigured
* Larger clusters need tuning (MTU, routing tables)

---

## **4. Difference between CNI and kube-proxy**

| CNI                                                  | kube-proxy                              |
| ---------------------------------------------------- | --------------------------------------- |
| Provides network connectivity & IP addresses to pods | Handles service networking              |
| L3 routing                                           | L4 load balancing                       |
| Responsible for pod-to-pod networking                | Forwards traffic to correct backend pod |
| Plugins: AWS VPC CNI, Calico, Cilium                 | Modes: iptables, IPVS                   |

---

## **5. How do you import an AWS resource into Terraform state?**

```bash
terraform import <resource_type.resource_name> <resource_id>
```

Example:

```bash
terraform import aws_instance.web i-0123456789
```

---

## **6. How do you delete a resource accidentally created by Terraform?**

**Two options:**

1. **Delete through Terraform**

   ```bash
   terraform destroy -target=resource_type.resource_name
   ```
2. **Or remove it from code and run:**

   ```bash
   terraform apply
   ```

---

## **7. How do you import infrastructure into Terraform if it was created manually?**

1. Write the resource block inside Terraform code **exactly matching** the existing infra.
2. Run:

   ```bash
   terraform import <resource> <id>
   ```
3. Run `terraform plan` to align differences.
4. Update code until plan is empty (in sync).

---

## **8. How do you upgrade Kubernetes?**

**Short practical answer:**

* **Upgrade control plane** (EKS version, GKE version, kubeadm upgrade).
* Drain & upgrade worker nodes:

  ```bash
  kubectl drain <node> --ignore-daemonsets
  ```
* Upgrade node AMIs or nodegroups (EKS managed node group upgrade).
* Rejoin nodes.
* Verify cluster health.

---

## **9. How do you check logs in Argo CD when a deployment fails?**

**Quick answer:**

* Check application events:

  ```bash
  kubectl describe pod <pod>
  ```
* Check ArgoCD controller logs:

  ```bash
  kubectl logs -n argocd deploy/argocd-application-controller
  ```
* Check ArgoCD UI: **App → Events → Pod logs**
* Check sync errors in UI under **App → Status → Operation**.

---





