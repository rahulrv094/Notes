✅ **Add short “Quick Notes to Remember” after every answer**

# **Scenario 1: WAR File Deployment on AWS EC2**

**You are given a Java WAR file and asked to deploy it on AWS EC2. Infrastructure is not ready. Explain how you will build VPC, subnets, EC2, security groups, IAM roles, and deploy the application.
How will you later automate this end-to-end environment setup?**

### **Answer (Interview Style):**

If the infrastructure is not ready, I would follow a structured approach to build everything from scratch.

### **1. Build VPC & Networking**

* Create **VPC** (10.0.0.0/16) with DNS support enabled.
* Create **2 public subnets** (for ALB/NAT/Bastion) and **2 private subnets** (for EC2/App).
* Attach an **Internet Gateway** to the VPC.
* Create **route tables**:

  * Public → IGW
  * Private → NAT Gateway (for outbound internet)

### **2. Security Groups**

* **App EC2 Security Group**

  * Inbound: 22 (SSH), 8080/80 (Tomcat)
  * Outbound: Allow all (or restrict based on compliance)
* (Optional) **ALB SG** to allow 80/443 from internet and forward to App SG.

### **3. IAM Roles**

* Create an **IAM Instance Profile** for EC2.
* Policies:

  * Read access to S3 (for WAR file)
  * CloudWatch logging/metrics
  * SSM (optional)

### **4. Create EC2 Instance**

* Choose Amazon Linux / Ubuntu in private subnet.
* Attach App SG + IAM role.
* In **User Data** install Java + Tomcat and deploy the WAR:

```bash
#!/bin/bash
yum update -y
yum install -y java-11-openjdk

cd /opt
wget https://downloads.../apache-tomcat-9.x.tar.gz
tar xzf apache-tomcat-9.x.tar.gz
ln -s apache-tomcat-9.x tomcat

aws s3 cp s3://bucket/app.war /opt/tomcat/webapps/app.war
/opt/tomcat/bin/startup.sh
```

### **5. Test the deployment**

* Access via EC2 public IP or ALB:
  `http://ALB-DNS/app`

---

### **How will you automate this later?**

* Use **Terraform** to automate VPC, subnets, SGs, EC2, IAM roles, ALB.
* Use **Packer** to build an AMI with Java + Tomcat pre-installed.
* Use **CI/CD (Jenkins/GitHub Actions)** to deploy new WAR automatically.
* Maintain different *tfvars* files for dev/stg/prod.

---

### **Quick Notes to Remember**

* VPC → Subnets → IGW → NAT → Route Tables
* SG → EC2 → IAM → User Data to deploy WAR
* Automate using Terraform + Packer + CI/CD

---

# **Scenario 2: Application Already Running – Introduce Monitoring**

**Your EC2 instance is running the application. Explain how you will introduce monitoring using Prometheus and Grafana.
How do you configure exporters and connect Grafana to Prometheus?**

### **Answer (Interview Style):**

### **1. Setup Prometheus**

* Deploy Prometheus on a separate EC2 instance or container.
* Configure its `prometheus.yml` file to define scrape jobs.

### **2. Install Exporters**

**Node Exporter** for OS-level metrics:

* CPU, memory, disk I/O, network
* Exposes metrics on `9100/metrics`

**JMX / Java Exporter** for app-level metrics:

* JVM memory, threads, GC
* Deploy using:
  `-javaagent:jmx_prometheus_javaagent.jar=9404:config.yaml`

### **3. Configure Prometheus Scraping**

In `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['10.0.1.9:9100']

  - job_name: 'java'
    static_configs:
      - targets: ['10.0.1.9:9404']
```

### **4. Setup Grafana**

* Install Grafana.
* Add **Prometheus as a Data Source**:

  * URL: `http://<prometheus-ip>:9090`
* Import built-in dashboards:

  * Node Exporter dashboard
  * JVM dashboard

---

### **Quick Notes to Remember**

* Install Node Exporter + JMX Exporter
* Prometheus scrapes → Grafana visualizes
* Grafana Data Source → Prometheus URL

---

# **Scenario 3: Custom Application-Level Metrics**

**Your manager wants custom application metrics such as request count, response size, scheduler events, etc. Explain how you will expose custom metrics for Prometheus.
How will Prometheus scrape these metrics and how will Grafana visualize them?**

### **Answer (Interview Style):**

### **1. Add Prometheus Client Library in Java**

* Use **Prometheus Java Client** or **Micrometer+Prometheus**.
* Expose metrics via `/metrics` endpoint.

### **2. Create Custom Metrics**

Example:

```java
static final Counter requestCount = Counter.build()
    .name("http_requests_total")
    .help("Total HTTP Requests")
    .labelNames("method","status")
    .register();

static final Summary responseSize = Summary.build()
    .name("response_size_bytes")
    .help("Response size")
    .register();
```

### **3. Expose Metrics Endpoint**

Start HTTP server:

```java
new HTTPServer(9404);
```

### **4. Prometheus Scraping**

Add:

```yaml
- job_name: 'custom_app'
  static_configs:
    - targets: ['10.0.1.9:9404']
```

### **5. Grafana Dashboards**

Use PromQL:

* Request rate:
  `sum(rate(http_requests_total[5m])) by (method, status)`
* Avg response size:
  `rate(response_size_bytes_sum[5m]) / rate(response_size_bytes_count[5m])`

---

### **Quick Notes to Remember**

* Integrate Prometheus client library
* Expose `/metrics`
* Prometheus scrapes → Grafana visualizes using PromQL

---

# **Scenario 4: PagerDuty Alert Response**

**You are the on-call engineer and PagerDuty alerts you about a CPU spike. Explain your incident response workflow.
What do you do if you cannot fix the alert yourself?**

### **Answer (Interview Style):**

### **1. Acknowledge the alert**

* Stop escalation chain while you work on it.

### **2. Investigate**

* Check Grafana dashboards (CPU, memory, request rate).
* SSH or SSM into EC2:

  * `top`, `htop`, `ps aux`
* Check logs around alert time.

### **3. Immediate Mitigation**

* Scale out using Auto Scaling Group.
* Restart application (if allowed).
* Roll back recent release if CPU spike started after deployment.

### **4. Long-term fix**

* Identify root cause (GC, infinite loop, memory leak).
* Create bug ticket.

### **5. If you cannot fix it**

* Escalate using **PagerDuty escalation policy**.
* Add notes on what you investigated.
* Hand over clearly with logs and timestamps.

---

### **Quick Notes to Remember**

* Acknowledge → Investigate → Mitigate → Escalate → Document

---

# **Scenario 5: Grafana Alerts Integrated with PagerDuty**

**You have Grafana dashboards and alerts created. Explain how you will integrate Grafana with PagerDuty.
What configurations are required in PagerDuty and Grafana?**

### **Answer (Interview Style):**

### **1. PagerDuty Setup**

* Create a **Service** (e.g., “Production App”).
* Add **Integration** → choose **Events API v2**.
* Copy the **Integration Key**.

### **2. Grafana Setup**

* Go to **Alerting → Contact Points**.
* Add contact type **PagerDuty**.
* Paste the Integration Key.
* Configure routing rules (e.g., severity=critical → PD).

### **3. Alerts Flow**

* Grafana alert triggers → sends event to PD → PD creates incident.

---

### **Quick Notes to Remember**

* PagerDuty: service + integration key
* Grafana: contact point + routing
* Alerts flow: Grafana → PD → on-call

---

# **Scenario 7: Infrastructure Automation Using Terraform**

**You need to create EC2, security groups, IAM roles using Terraform. Explain Terraform lifecycle and execution steps.**

### **Answer (Interview Style):**

### **Terraform Lifecycle**

1. **Write** `.tf` files
2. **Init** → downloads providers/modules
3. **Plan** → shows what will be created/modified
4. **Apply** → executes changes
5. **Destroy** → deletes resources

### **Execution Steps**

* `terraform init`
* `terraform fmt` & `terraform validate`
* `terraform plan -var-file=dev.tfvars`
* `terraform apply -var-file=dev.tfvars`
* Remote state stored in S3 + DynamoDB lock

---

### **Quick Notes to Remember**

* Init → Plan → Apply → Destroy
* Use remote backend
* Use modules for reusability

---

# **Scenario 8: Jenkins Deployment on AWS**

**You have Jenkins pipeline, Terraform code, and Dockerfile. Explain how Jenkins triggers infrastructure creation and deploys the application.
How do you manage credentials, environments, and deployment order?**

### **Answer (Interview Style):**

### **Pipeline Flow**

1. **Checkout code**
2. **Build & Test** using Maven/Gradle
3. **Build Docker image** and push to ECR
4. **Terraform apply** to create/update infra
5. **Deploy container** to EC2/ECS/EKS
6. **Post-deploy tests**

### **Managing Credentials**

* Store AWS creds, GitHub token, Docker creds in Jenkins Credential Store
* Use `withCredentials` block
* Use Role Assumption for prod

### **Managing Environments**

* Separate `dev`, `stg`, `prod` workspaces
* Different tfvars for each environment
* Include a manual approval step before prod

### **Deployment Order**

* Build → Infra → App Deploy
* Use Blue-Green/Canary for safe rollout

---

### **Quick Notes to Remember**

* Jenkins triggers Terraform → then deploy
* Credentials via Jenkins store
* Separate workspaces/env configs

---

# **Scenario 9: Application Logs & Observability**

**Explain how you ship application logs to Splunk or Datadog.
What dashboards and alerts do you configure for SRE observability?**

### **Answer (Interview Style):**

### **1. Shipping Logs**

* **For Splunk**:

  * Install **Splunk Universal Forwarder** on EC2
  * Configure inputs for app logs
* **For Datadog**:

  * Install Datadog Agent
  * Enable log collection in `datadog.yaml`

### **2. Centralized Logging via CloudWatch**

* App logs → CloudWatch
* Use Splunk/Datadog forwarder to ingest logs

### **3. Dashboards**

SRE Golden Signals:

* Latency (p95, p99)
* Error rate
* Traffic (RPS)
* Saturation (CPU, memory, threads)

### **4. Alerts**

* High latency
* High error rate
* Slow DB queries
* No logs incoming (service crash)
* Disk full, CPU spike, memory pressure

---

### **Quick Notes to Remember**

* Use agents or CloudWatch forwarders
* Monitor golden signals
* Alerts = latency + errors + saturation

---

# **Scenario 10: High Availability & Reliability**

**You are asked to design infrastructure with high availability, security, and failover. What factors do you consider?
How do you ensure reliability from an SRE perspective?**

### **Answer (Interview Style):**

### **1. High Availability**

* Multi-AZ deployment
* Auto Scaling Group
* ALB for traffic distribution
* Multi-AZ RDS or DynamoDB

### **2. Security**

* Private subnets for app & DB
* Least privilege IAM roles
* Security Groups + NACLs
* Secrets Manager for credentials

### **3. Failover & DR**

* Automated backups
* Cross-region replication
* DR Runbooks
* Defined RTO/RPO

### **4. SRE Reliability Practices**

* Define SLOs & SLIs
* Manage error budgets
* Run chaos engineering tests
* Use safe deployments (canary/blue-green)
* Blameless postmortems

---

### **Quick Notes to Remember**

* Multi-AZ, ASG, ALB
* Security: SGs, IAM, Secrets Manager
* Reliability: SLOs, error budgets, runbooks

---

# **11. Why Terraform instead of Ansible?**

### **Answer (Interview Style):**

* Terraform is **declarative** and manages **infrastructure provisioning**.
* Maintains **state**, understands resource relationships.
* Creates dependency graph and parallelizes creation.
* Ansible is mainly for **configuration management**.
* Terraform is better for VPC, subnets, EC2, ALB, IAM etc.

### **Quick Notes**

* Terraform = infra provisioning
* Ansible = config management
* Terraform has state + dependency graph

---

# **12. Why Terraform instead of Python?**

### **Answer (Interview Style):**

* Terraform is purpose-built for Infrastructure as Code.
* Provides **plan → apply** safety.
* Maintains **state** and detects drift.
* Has huge module ecosystem.
* Python would require custom code for everything (SDK calls with no state mgmt).
* Terraform is easier for teams to review, maintain, and scale.

### **Quick Notes**

* Terraform has state, plan, modules
* Python = manual, procedural, no infra-state

---

If you want, I can convert this entire Q&A into:

📄 **PDF**
📘 **Markdown file**
🎓 **Interview revision sheet**
📝 **One-page cheat-sheet**

Just tell me the format you want.
