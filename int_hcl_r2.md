Q1. Scenario 1 – WAR File Deployment on AWS EC2
**You are given a Java WAR file and infra is not ready. How will you build VPC, subnets, EC2, SGs, IAM roles and deploy? How will you automate later?**

**Answer:**

1. **Design the basic architecture**

* 1 VPC (e.g. `10.0.0.0/16`)
* 2 public subnets (for bastion / NAT / ALB) – in 2 AZs
* 2 private subnets (for app EC2, DB later) – in 2 AZs
* 1 Internet Gateway attached to the VPC
* Route tables:

  * Public route table: route `0.0.0.0/0` → Internet Gateway
  * Private route table: route `0.0.0.0/0` → NAT Gateway (optional for outbound internet)

2. **Create networking components**

* Create **VPC** with DNS support enabled.
* Create **subnets**:

  * `public-subnet-a`, `public-subnet-b`
  * `private-subnet-a`, `private-subnet-b`
* Attach an **Internet Gateway** and update public route table.
* (Optional) Create **NAT Gateway** in one public subnet for private instances’ outbound internet.

3. **Security Groups**

* **App SG** (for EC2 with WAR):

  * Inbound:

    * Allow 22 (SSH) from office IP / bastion SG
    * Allow 8080 (Tomcat) or 80/443 from ALB SG or your IP (for POC)
  * Outbound: allow all (or restricted as per policy)
* (Optional) **ALB SG**:

  * Inbound: 80/443 from internet
  * Outbound: 8080/80/443 to App SG

4. **IAM Roles**

* **Instance role** for EC2 (via IAM instance profile):

  * Permissions for:

    * Reading from S3 (if WAR is stored there)
    * CloudWatch logs/metrics (if needed)
* Follow **least privilege** (no full admin).

5. **Launch EC2**

* Choose Amazon Linux 2 / Ubuntu.

* Place EC2 in **private subnet** (better) or public subnet (for simple demo).

* Attach **App SG** and **IAM Role**.

* Use **User Data** to bootstrap:

  ```bash
  #!/bin/bash
  yum update -y
  yum install -y java-11-openjdk wget
  # Install Tomcat
  cd /opt
  wget https://downloads.apache.org/tomcat/.../apache-tomcat-9.x.tar.gz
  tar xzf apache-tomcat-9.x.tar.gz
  ln -s apache-tomcat-9.x tomcat
  # Deploy WAR (from S3)
  aws s3 cp s3://my-bucket/app.war /opt/tomcat/webapps/app.war
  # Start Tomcat
  /opt/tomcat/bin/startup.sh
  ```

* Verify `http://<ec2-ip-or-alb>/app`.

6. **How will you later automate this end-to-end setup?**

* Use **Terraform** for infra (VPC, subnets, IGW, SGs, IAM roles, EC2, ALB).

* Use **modules**:

  * `vpc/`, `networking/`, `security/`, `compute/`

* Infra flow:

  ```bash
  terraform init
  terraform plan -var-file=env/dev.tfvars
  terraform apply -var-file=env/dev.tfvars
  ```

* Use **User Data** or **Ansible** to configure EC2 (install Java, Tomcat, deploy WAR) OR build **AMI** with Packer and just launch that AMI.

* Wire this into CI/CD (Jenkins/GitHub Actions) so env creation and deployment are **repeatable** from code.

---

Q2. Scenario 2 – Application Already Running – Introduce Monitoring
**EC2 already running the app. How will you introduce Prometheus and Grafana? How to configure exporters and connect Grafana to Prometheus?**

**Answer:**

1. **Components**

* **Prometheus server** – scrapes metrics.
* **Exporters** – expose metrics in `/metrics` endpoint:

  * `node_exporter` for OS / host metrics (CPU, memory, disk, network).
  * `jmx_exporter` or Java client library for JVM/app metrics.
* **Grafana** – visualizes Prometheus data.

2. **Install exporters on EC2**

* **Node exporter**:

  ```bash
  useradd --no-create-home --shell /bin/false node_exporter
  wget https://github.com/prometheus/node_exporter/...tar.gz
  tar xzf node_exporter-*.tar.gz
  cp node_exporter-*/node_exporter /usr/local/bin/
  ```

  Create systemd service and start it. It exposes metrics on `:9100/metrics`.

* **JMX exporter / app exporter**:

  * For Java, either:

    * Run JVM with `-javaagent:/path/jmx_prometheus_javaagent.jar=9404:config.yaml`
    * Or integrate Prometheus Java client and expose `/metrics` via HTTP.

3. **Prometheus setup**

* Deploy Prometheus on another EC2, container, or Kubernetes.

* In `prometheus.yml`, configure scrape jobs:

  ```yaml
  scrape_configs:
    - job_name: 'node'
      static_configs:
        - targets: ['10.0.1.10:9100']  # node_exporter

    - job_name: 'java_app'
      static_configs:
        - targets: ['10.0.1.10:9404']  # jmx exporter or /metrics
  ```

* Start Prometheus, verify `http://<prometheus-host>:9090`.

4. **Connect Grafana to Prometheus**

* Install Grafana (same host as Prometheus or separate).
* In Grafana UI:

  * **Configuration → Data sources → Add data source → Prometheus**
  * Set URL: `http://<prometheus-host>:9090`
  * Save & Test.
* Import **pre-built dashboards** for node_exporter / JVM or build custom panels using PromQL queries.

---

Q3. Scenario 3 – Custom Application-Level Metrics
**Manager wants metrics like request count, response size, scheduler events. How do you expose custom metrics, how Prometheus scrapes them, how Grafana visualizes them?**

**Answer:**

1. **Expose custom metrics from the Java app**

* Use **Prometheus Java client** (or Micrometer with Prometheus registry). Example with Java client:

  ```java
  import io.prometheus.client.Counter;
  import io.prometheus.client.Summary;
  import io.prometheus.client.exporter.HTTPServer;
  import io.prometheus.client.hotspot.DefaultExports;

  public class MetricsConfig {
      static final Counter requestCount = Counter.build()
          .name("http_requests_total")
          .help("Total HTTP requests.")
          .labelNames("method", "status")
          .register();

      static final Summary responseSize = Summary.build()
          .name("http_response_size_bytes")
          .help("Response size in bytes.")
          .register();

      public static void init() throws Exception {
          DefaultExports.initialize(); // JVM metrics
          new HTTPServer(9404); // exposes /metrics
      }
  }
  ```

* In your controllers/filters:

  ```java
  requestCount.labels(method, status).inc();
  responseSize.observe(responseBytes);
  ```

* Similarly for **scheduler events**:

  ```java
  static final Counter schedulerEvents = Counter.build()
      .name("scheduler_events_total")
      .help("Total scheduled job runs.")
      .labelNames("job_name", "status")
      .register();
  ```

2. **How will Prometheus scrape these metrics?**

* App now exposes `/metrics` on `:9404` (or via JMX exporter).

* In `prometheus.yml`, add job:

  ```yaml
  - job_name: 'custom_app_metrics'
    static_configs:
      - targets: ['10.0.1.10:9404']
  ```

* Prometheus periodically (e.g. every 15s) hits `/metrics`, parses all the custom metrics.

3. **How will Grafana visualize them?**

* Create new dashboard in Grafana.

* Example panels:

  * **Request rate:**

    PromQL: `sum(rate(http_requests_total[5m])) by (method, status)`

  * **Top response size:**

    `sum(rate(http_response_size_bytes_sum[5m])) / sum(rate(http_response_size_bytes_count[5m]))`

  * **Scheduler events:**

    `sum(rate(scheduler_events_total[5m])) by (job_name, status)`

* Use graphs, tables, heatmaps for latency, and set **thresholds** and **alerts**.

---

Q4. Scenario 4 – PagerDuty Alert Response
**On-call, PagerDuty alerts about CPU spike. What’s your workflow? What if you can’t fix it?**

**Answer:**

1. **Immediate steps**

* **Acknowledge** alert in PagerDuty (stops further escalation temporarily).
* Check **Grafana** / monitoring dashboard:

  * CPU %, load average, memory, disk IO.
  * Correlate with **request rate, error rate, latency**.
* Identify **scope**: one instance or many? One service or multiple?

2. **Quick triage on EC2**

* SSH to instance (if allowed) or use SSM.

* Check processes:

  ```bash
  top, htop, ps aux --sort=-%cpu | head
  ```

* Look for:

  * High CPU from Java process → maybe GC, loops.
  * High CPU from rogue process / cronjob.

* Check logs around alert time for errors / spikes.

3. **Short-term mitigation**

* If app is overloaded:

  * **Scale out** – increase EC2 count via ASG or add instances behind ALB.
  * Temporarily **increase instance size** (depending on policy).
* If bug / runaway process:

  * Restart service (if allowed by runbook).
  * Roll back recent deployment if CPU spike started after release.
* If DB / dependency is bottleneck, apply **rate limiting** or temporary **feature flag**.

4. **If you cannot fix it yourself**

* Follow **runbook / escalation policy**:

  * Add notes in PagerDuty incident: what you checked, logs, graphs, timestamps.
  * **Escalate** to next level: SRE L2 / application team via PagerDuty escalation chain.
  * Keep communication clear in Slack / Teams / incident channel.
* Don’t silently wait; your job is **triage + stabilize + escalate** with context.

5. **Post-incident**

* Participate in **blameless postmortem**: root cause, timeline, what worked/failed.
* Add or update **runbooks**, alerts, dashboards so next time is faster.
* Possibly add new **metrics** (e.g. GC time, queue length) if missing.

---

Q5. Scenario 5 – Grafana Alerts Integrated with PagerDuty
**You have Grafana dashboards + alerts. How to integrate with PagerDuty? What configs in PagerDuty and Grafana?**

**Answer:**

1. **PagerDuty configuration**

* Create / use a **Service** in PagerDuty (e.g. `my-app-prod`).
* Under that service, create an **Integration**:

  * Choose type: **Events API v2** (or Grafana integration if available).
  * PagerDuty gives you an **Integration Key**.
* Set **Escalation Policy** and **Notification Rules** for that service.

2. **Grafana configuration**

* In Grafana:

  * Go to **Alerting → Contact points**.
  * Add new contact point type **PagerDuty**.
  * Paste the **Integration Key**.
  * Optionally map **severity** (e.g. labels from alert to PD severity).
* Create **Notification Policies** in Grafana alerting:

  * Route alerts with label `severity="critical"` to PagerDuty contact point.
* Ensure **alert rules** (for CPU, latency, errors) have those labels.

3. **Flow**

* Grafana alert rule fires → sends event with payload to PagerDuty Events API using key → PagerDuty creates incident → notifies on-call via configured channels.

---

Q7. Scenario 7 – Infrastructure Automation Using Terraform
**You need to create EC2, SGs, IAM using Terraform. Explain Terraform lifecycle and execution steps.**

**Answer:**

1. **Terraform basic lifecycle**

* **Write** code: define resources in `.tf` files.
* **Initialize**: `terraform init` – download providers, modules.
* **Plan**: `terraform plan` – show what Terraform will create/change/destroy.
* **Apply**: `terraform apply` – actually perform changes.
* **Destroy** (when needed): `terraform destroy` – tear down infra.

2. **Code structure**

* `providers.tf` – AWS region, provider configuration.
* `variables.tf` – define variables (e.g. instance_type).
* `main.tf` – resources:

  * `aws_vpc`, `aws_subnet`, `aws_security_group`, `aws_iam_role`, `aws_instance`.
* `outputs.tf` – expose outputs: instance IP, ALB DNS, etc.
* Use **modules** for reusability (e.g. module for EC2 + SG).

3. **Example EC2 + SG snippet**

```hcl
resource "aws_security_group" "app_sg" {
  name        = "app-sg"
  description = "App SG"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "app" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.private_a.id
  vpc_security_group_ids = [aws_security_group.app_sg.id]
  iam_instance_profile   = aws_iam_instance_profile.app_profile.name
  user_data              = file("user-data.sh")
}
```

4. **Execution steps**

* `terraform init`
* `terraform fmt && terraform validate`
* `terraform plan -var-file=env/dev.tfvars`
* Review plan, then `terraform apply -var-file=env/dev.tfvars`.
* For CI/CD: run same steps in Jenkins, store **state** in **remote backend** (S3 + DynamoDB lock).

---

Q8. Scenario 8 – Jenkins Deployment on AWS
**You have Jenkins pipeline, Terraform, Dockerfile. How does Jenkins trigger infra creation and deploy app? How manage credentials, envs, and order?**

**Answer:**

1. **High-level pipeline stages**

* **Stage 1 – Checkout**:

  * Pull code repo (app code, Dockerfile, Terraform modules).

* **Stage 2 – Build & Test**:

  * `mvn test` / `gradle test`.
  * Static code analysis if needed.

* **Stage 3 – Build Docker Image**:

  ```bash
  docker build -t my-app:${GIT_COMMIT} .
  docker tag my-app:${GIT_COMMIT} <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-app:${GIT_COMMIT}
  docker push <ecr-url>/my-app:${GIT_COMMIT}
  ```

* **Stage 4 – Terraform Infra**:

  * `terraform init`
  * `terraform plan`
  * `terraform apply` to create/update EC2/ALB/SGs, etc.
  * Use **workspace** or `-var-file` per environment (`dev`, `stg`, `prod`).

* **Stage 5 – Deploy Application**:

  * If using EC2:

    * SSH / Ansible to pull Docker image from ECR and run container.
  * If using ECS/EKS:

    * Update ECS task definition or run `kubectl set image` for deployment.

2. **Manage credentials**

* Store AWS credentials / tokens in Jenkins **Credentials** store:

  * AWS Access Key / Secret Key or assume **IAM role** via `aws sts assume-role`.
  * Docker registry/ECR credentials (or use ECR login in pipeline).
* Use **withCredentials** block in Jenkinsfile to inject them as env vars.
* Never hardcode secrets in repo.

3. **Manage environments**

* Separate **Jenkins jobs / pipelines** per env or use parameters (`ENV=dev/stg/prod`).
* Use **Terraform workspaces** or different state backends per env.
* Use separate **AWS accounts** or at least separate VPCs for each environment.

4. **Deployment order / safety**

* Order typically:

  1. Build & test app
  2. Build & push Docker image
  3. Apply Terraform to ensure infra is ready
  4. Update service (ECS/EKS/EC2) to new image
* For prod, use **manual approval** step after staging deploy.
* Prefer **blue/green** or **rolling** deployments to avoid downtime.

---

Q9. Scenario 9 – Application Logs & Observability
**How do you ship logs to Splunk/Datadog? What dashboards and alerts for SRE observability?**

**Answer:**

1. **Shipping logs**

* **Option 1 – Agent on EC2**:

  * Install **Splunk Universal Forwarder** or **Datadog Agent**.
  * Configure log paths: `/var/log/app/*.log`, `/var/log/nginx/*.log`.
* **Option 2 – CloudWatch Logs → Forwarder**:

  * App writes logs to **CloudWatch Logs**.
  * Use Splunk/Datadog **CloudWatch integration** or Lambda forwarder to ship logs.
* Use **structured logging** (JSON) with fields: `timestamp`, `level`, `service`, `request_id`, `user_id`, `latency_ms`.

2. **Dashboards for SRE observability**

At minimum, have 4 “golden signals”:

* **Latency**:

  * p50, p95, p99 request latency per endpoint.
* **Traffic**:

  * Requests per second, per service / route.
* **Errors**:

  * HTTP 5xx, 4xx counts, error logs by type.
* **Saturation**:

  * CPU, memory, thread pool, connection pool utilization.

Other useful views:

* Top error messages / stack traces.
* Log volume per service.
* Slow queries (DB logs).
* Correlation by **trace id** if using distributed tracing.

3. **Alerts**

* Error rate > X% for N minutes.
* p95 latency above threshold for N minutes.
* No logs (or drastic drop) from service (indicates crash).
* Infrastructure alerts: disk full, memory pressure, CPU throttle.
* SLO-based alerts:

  * Example: 99.9% requests must be < 300 ms; alert when burning error budget fast.

---

Q10. Scenario 10 – High Availability & Reliability
**Design infra with HA, security, failover. What factors? How to ensure reliability from SRE view?**

**Answer:**

1. **High availability design factors**

* **Multi-AZ**:

  * EC2 instances in at least **2 AZs** behind an **ALB**.
  * DB (RDS) with **Multi-AZ**.
* **Auto Scaling**:

  * ASG for app tier with health checks.
* **Stateless app**:

  * Store session in Redis/ElastiCache, not on instance.
* **Resilient storage**:

  * S3 for assets, EFS if shared filesystem needed.

2. **Security**

* **Network segmentation**:

  * Public subnets for ALB/NAT; private for app/DB.
* **Security Groups / NACLs**:

  * Principle of least access (only necessary ports).
* **IAM**:

  * Least-privilege roles for EC2, ECS, Lambda.
* **Secrets management**:

  * AWS Secrets Manager / SSM Parameter Store, not plain text.

3. **Failover / DR**

* **Backups & snapshots** of DBs and configurations.
* Define **RTO** (Recovery Time Objective) and **RPO** (Recovery Point Objective).
* Possibly **multi-region DR**:

  * Replicated data (e.g., cross-region read replica).
  * DR runbooks to spin up infra in backup region.

4. **SRE reliability practices**

* Define **SLOs** (Service Level Objectives):

  * Availability (e.g. 99.9%) and latency goals.
* Maintain **error budgets** and use them to control release velocity.
* Have **playbooks/runbooks** for common incidents.
* **Automated tests** + CI/CD with safe deployment strategies (blue/green, canary).
* **Chaos testing** / game days to ensure system really tolerates failures.
* Continuous improvement via **postmortems**.

---

Q11. Why Terraform instead of Ansible?

**Answer:**

* **Purpose**:

  * **Terraform**: designed for **infrastructure provisioning** (VPC, subnets, EC2, RDS, ALB, etc.).
  * **Ansible**: mainly for **configuration management** and **orchestration** (install packages, edit config files, deploy apps).

* **State management**:

  * Terraform maintains **state** (what resources exist) and calculates **diff** between desired and actual.
  * Ansible is mostly **stateless** (you run playbooks; it doesn’t keep a full graph of resources in a state file).

* **Declarative vs procedural**:

  * Terraform is **declarative**: “I want N instances with these properties.”
  * Ansible is more **imperative/procedural**: step-by-step tasks (though you can make them idempotent).

* **Dependency graph**:

  * Terraform builds a **resource graph**, can create resources in parallel respecting dependencies.
  * Ansible executes tasks mostly in order; dependencies are manual.

* **Multi-cloud support & ecosystem**:

  * Terraform has a huge set of **providers** for AWS, GCP, Azure, Datadog, GitHub, etc.
  * While Ansible has modules, Terraform is more standard for IaC provisioning.

**Short answer:**
Use Terraform to **create** and manage cloud infrastructure as code with proper state and planning; use Ansible to **configure** what runs on that infrastructure. They complement each other, not compete.

---

Q12. Why Terraform instead of Python?

**Answer:**

* **Standardization**:

  * Terraform is a **tool + DSL** (HCL) built specifically for infra.
  * Python is a general-purpose language; you would have to write a lot of custom code or use SDKs (boto3) and reinvent planning, diff, state, modules.

* **Planning and safety**:

  * Terraform has `plan` → shows exactly what it will change before apply.
  * With Python scripts, you typically don’t get an automatic “plan”; you must implement your own dry-run logic.

* **State & drift management**:

  * Terraform keeps **state** and can detect **drift**.
  * Python scripts usually perform API calls without centralized state unless you build it.

* **Modules and ecosystem**:

  * Terraform has huge **module registry** (VPCs, EKS, RDS, etc.).
  * With Python, you need to write or maintain more code yourself.

* **Team readability**:

  * HCL is easy for infra / DevOps teams to read and review.
  * Python infra scripts can become complex and vary a lot in style.

**Short answer:**
Terraform gives you **declarative infra-as-code with state, plan/apply, ecosystem, and team-standard workflows** out of the box. Python is powerful but would require you to **build your own Terraform** features, which is extra work and more risk.

---

If you want, I can also turn this into a one-page **interview cheat sheet** with bullet answers you can revise quickly before an SRE/DevOps round.
