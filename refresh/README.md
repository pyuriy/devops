# DevOps Refresh

DevOps is an approach to software development that combines development (Development) and operations (Operations) into a single unified methodology. The primary goal of DevOps is to increase the efficiency and speed of software development through process automation, improved communication, and collaboration between different teams within the development and operations framework.

---

## (Linux)[Linux]
*   **Refresh your knowledge** of Unix-like system administration by installing custom services and deploying Redis, Nginx, RabbitMQ, PostgreSQL, MySQL, WireGuard VPN, etc.
*   **Learn to write automation** in Bash and utilize Docker.

## CI/CD
*   **Master the basics** of version control using Git.
*   **Create a basic CI/CD pipeline** in GitLab.
*   **Integrate builds, tests, and Docker** into the pipeline, expanding automation capabilities for application development and releases.

## Terraform
Tasks to master the basics of Terraform for Infrastructure as Code (IaC) automation:
*   **Install Terraform CLI** and create a script to deploy an Nginx Docker container.
*   **Configure a CI/CD pipeline** with `terraform plan` in GitHub Actions.
*   **Refactor code into modules** while adhering to stylistic standards.
*   **Integrate TFLint** for code quality checks.
*   **Deploy a full-stack Docker application** featuring a frontend, backend, and Nginx as a reverse proxy with a self-signed SSL certificate, using the `hashicorp/tls` provider and a modular structure.

## AWS
*   **Deploy diverse resources**, ranging from basic EC2 and S3 to serverless applications on Lambda.
*   **Configure an application on EC2**, migrate it to ECS, and eventually make it entirely serverless.

## Ansible
Master the basics of Ansible for configuration automation:
*   **Create a Terraform script** to deploy an EC2 instance (or a VM via Vagrant).
*   **Develop an Ansible playbook** to install Nginx and Drone via `docker-compose` with Let’s Encrypt certificates and auto-renewal.
*   **Refactor the playbook** into modular roles.
*   **Configure dynamic EC2 inventory.**
*   **Add monitoring** via Prometheus, Grafana, and Alertmanager on separate instances using a `monitoring_install` role. This includes setting up Nginx as a reverse proxy, integrating CloudWatch Logs, automatically adding new instances to Prometheus, and (optionally) using the Nginx exporter for statistics collection—all managed via CI/CD in GitLab.

## Kubernetes
*   **Set up** a Minikube environment.
*   **Deploy an application.**
*   **Study complex Kubernetes objects** and best practices for working with Kubernetes.
*   **Learn to use Helm and Kustomize** for manifest reuse.

## Incident Management
*   **Explore different types of reliability** (Disaster Recovery, High Availability, Fault Tolerance) and define their application in real-world scenarios.
*   **Familiarize yourself with post-mortem culture**, including key aspects of incident management and reporting to improve processes, demonstrating a comprehensive understanding of the topic.
