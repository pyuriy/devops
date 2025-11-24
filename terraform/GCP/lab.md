# Comprehensive Terraform with GCP Lab

This hands-on lab guides you through provisioning infrastructure on Google Cloud Platform (GCP) using Terraform. You'll start with basics and build up to more advanced concepts like modules and remote state. By the end, you'll have a complete, deployable setup including a VPC, VM, storage bucket, and a simple GKE cluster—while learning best practices.

**Estimated Time:** 2-3 hours  
**Level:** Beginner to Intermediate  
**Cost Warning:** This lab creates billable resources (~$0.50-2/hour depending on region). Always monitor via [GCP Billing](https://console.cloud.google.com/billing). Use free tier where possible (e.g., e2-micro VM).

## Prerequisites

1. **GCP Account:** Sign up at [cloud.google.com](https://cloud.google.com) with billing enabled. Create a new project (e.g., `terraform-lab-2025`).
2. **Tools:**
   - [Terraform CLI](https://developer.hashicorp.com/terraform/install): Download and verify with `terraform version` (v1.9+ recommended).
   - [Google Cloud SDK](https://cloud.google.com/sdk/docs/install): Install `gcloud`. Authenticate: `gcloud auth login` and `gcloud auth application-default login`.
3. **Enable APIs:** Run `gcloud services enable compute.googleapis.com container.googleapis.com storage.googleapis.com sqladmin.googleapis.com iap.googleapis.com` (or enable via console).
4. **Local Setup:** Create a directory: `mkdir terraform-gcp-lab && cd terraform-gcp-lab`. Use VS Code or any editor.

**Pro Tip:** Fork this lab's code from [GitHub](https://github.com/example/terraform-gcp-lab) (imaginary; create your own repo) for version control.

## Lab Overview

We'll build:
- A VPC network with subnet and firewall.
- A Compute Engine VM.
- A Cloud Storage bucket.
- IAM service account for the VM.
- A basic GKE cluster using a module.
- Use variables, outputs, and remote state.

Run commands in your terminal from the lab directory.

---

## Exercise 1: Project Setup and Provider Configuration

**Goal:** Configure Terraform to interact with your GCP project.

1. **Set Project ID:** Export your project ID: `export TF_VAR_project_id="your-project-id"` (replace with actual ID from `gcloud projects list`).

2. **Create `main.tf`:** Paste this (updated for latest provider v7.11.0):
   ```hcl
   terraform {
     required_version = ">= 1.9"
     required_providers {
       google = {
         source  = "hashicorp/google"
         version = "~> 7.0"  # Latest as of Nov 2025
       }
     }
     # We'll add backend in Ex8
   }

   provider "google" {
     project = var.project_id
     region  = "us-central1"
     zone    = "us-central1-a"
   }

   variable "project_id" {
     type        = string
     description = "GCP Project ID"
   }
   ```

3. **Initialize:** `terraform init`. Output should show provider download.

4. **Validate:** `terraform validate`. Should succeed.

**Verification:** `terraform providers` lists `hashicorp/google`.

**Troubleshoot:** If auth fails, run `gcloud auth application-default print-access-token` and ensure no expired tokens.

---

## Exercise 2: Networking Basics (VPC, Subnet, Firewall)

**Goal:** Create a secure VPC for your resources.

1. **Add to `main.tf`** (or create `network.tf` for modularity):
   ```hcl
   resource "google_compute_network" "vpc" {
     name                    = "lab-vpc"
     auto_create_subnetworks = false
     mtu                     = 1460
   }

   resource "google_compute_subnetwork" "subnet" {
     name          = "lab-subnet"
     ip_cidr_range = "10.0.1.0/24"
     region        = "us-central1"
     network       = google_compute_network.vpc.id
     private_ip_google_access = true  # For private GKE
   }

   resource "google_compute_firewall" "allow_ssh_http" {
     name    = "allow-ssh-http"
     network = google_compute_network.vpc.name

     allow {
       protocol = "tcp"
       ports    = ["22", "80"]
     }

     source_ranges = ["0.0.0.0/0"]  # Tighten for prod (e.g., your IP)
   }
   ```

2. **Plan & Apply:** 
   - `terraform plan` (review: 3 resources to add).
   - `terraform apply --auto-approve`.

3. **Verify:** 
   - GCP Console > VPC Network > lab-vpc (check subnets/firewalls).
   - `terraform show` for state details.

**Output Example:**
```
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

**Best Practice:** Use descriptive names like `lab-{component}-{env}`.

---

## Exercise 3: Compute Engine VM

**Goal:** Launch a web server VM in your VPC.

1. **Add to `main.tf`** (or `compute.tf`):
   ```hcl
   resource "google_compute_instance" "web-vm" {
     name         = "lab-web-vm"
     machine_type = "e2-micro"  # Free tier eligible
     zone         = "us-central1-a"

     boot_disk {
       initialize_params {
         image = "debian-cloud/debian-12"  # Latest Debian
         size  = 10
         type  = "pd-standard"
       }
     }

     network_interface {
       network = google_compute_network.vpc.id
       subnetwork = google_compute_subnetwork.subnet.id
       access_config {
         # Ephemeral public IP
       }
     }

     metadata = {
       startup-script = <<-EOF
         #!/bin/bash
         apt-get update
         apt-get install -y apache2
         echo "<h1>Hello from Terraform Lab!</h1>" > /var/www/html/index.html
         systemctl restart apache2
       EOF
     }

     tags = ["http-server"]  # For firewall
     service_account {
       # We'll assign in Ex5
       scopes = ["cloud-platform"]
     }
   }

   output "vm_public_ip" {
     value = google_compute_instance.web-vm.network_interface[0].access_config[0].nat_ip
   }
   ```

2. **Apply:** `terraform apply`. Wait ~2 min.

3. **Verify:**
   - Output: `terraform output vm_public_ip` (note IP).
   - Browser: `http://<IP>` (see "Hello from Terraform Lab!").
   - SSH: `gcloud compute ssh lab-web-vm --zone=us-central1-a`.

**Troubleshoot:** Firewall blocking? Check tags match. No IP? Ensure `access_config`.

**Extension:** Add `lifecycle { prevent_destroy = true }` to protect the VM.

---

## Exercise 4: Cloud Storage Bucket

**Goal:** Create a versioned bucket for static files.

1. **Add to `main.tf`** (or `storage.tf`):
   ```hcl
   resource "google_storage_bucket" "lab-bucket" {
     name                        = "tf-lab-bucket-${random_id.bucket_suffix.hex}"  # Unique
     location                    = "US"
     force_destroy               = true
     uniform_bucket_level_access = true

     versioning {
       enabled = true
     }

     labels = {
       lab = "terraform"
     }
   }

   resource "random_id" "bucket_suffix" {
     byte_length = 4
   }

   output "bucket_name" {
     value = google_storage_bucket.lab-bucket.name
   }
   ```

2. **Apply:** `terraform apply`.

3. **Verify:**
   - `terraform output bucket_name`.
   - GCP Console > Storage > Upload a file.
   - CLI: `gsutil mb -p $TF_VAR_project_id gs://<bucket-name>` (but Terraform already did it).

**Best Practice:** Use `random_id` for uniqueness; enable versioning for recovery.

---

## Exercise 5: IAM and Service Accounts

**Goal:** Securely assign roles to your VM's service account.

1. **Add to `main.tf`** (or `iam.tf`):
   ```hcl
   resource "google_service_account" "vm-sa" {
     account_id   = "lab-vm-sa"
     display_name = "Lab VM Service Account"
   }

   resource "google_project_iam_member" "sa-roles" {
     for_each = toset(["roles/compute.instanceAdmin.v1", "roles/storage.objectViewer"])

     project = var.project_id
     role    = each.key
     member  = "serviceAccount:${google_service_account.vm-sa.email}"
   }

   # Attach to VM (update existing resource)
   resource "google_compute_instance" "web-vm" {
     # ... (previous config)
     service_account {
       email  = google_service_account.vm-sa.email
       scopes = ["cloud-platform"]
     }
     # ...
   }
   ```

2. **Apply:** `terraform apply` (updates VM without recreation).

3. **Verify:** GCP Console > IAM > Search `lab-vm-sa` (check roles). Test: SSH to VM and run `gsutil ls` (lists buckets if scoped).

**Security Tip:** Principle of least privilege—only assign needed roles.

---

## Exercise 6: Using Modules (GKE Cluster)

**Goal:** Deploy a managed Kubernetes cluster using an official module.

1. **Download Module:** No download needed—Terraform pulls from registry.

2. **Add to `main.tf`** (or `gke.tf`):
   ```hcl
   module "gke" {
     source  = "terraform-google-modules/kubernetes-engine/google"
     version = "~> 30.0"  # Check latest at registry

     project_id = var.project_id
     name       = "lab-gke-cluster"
     region     = "us-central1"
     network    = google_compute_network.vpc.name
     subnetwork = google_compute_subnetwork.subnet.name

     # Basic node pool
     node_pools = [
       {
         name         = "default-pool"
         machine_type = "e2-medium"
         min_count    = 1
         max_count    = 3
         disk_size_gb = 100
       }
     ]

     # Enable autopilot for simplicity (optional)
     enable_autopilot = false
   }

   output "gke_cluster_endpoint" {
     value = module.gke.endpoint
   }
   ```

3. **Apply:** `terraform plan` (expect ~10 resources). `terraform apply`.

4. **Verify:**
   - `terraform output gke_cluster_endpoint`.
   - `gcloud container clusters get-credentials lab-gke-cluster --region us-central1`.
   - `kubectl get nodes` (connect to cluster).

**Pro Tip:** Modules reduce boilerplate. Explore [Terraform GCP Modules](https://registry.terraform.io/modules/terraform-google-modules).

**Troubleshoot:** API not enabled? Run `gcloud services enable container.googleapis.com`.

---

## Exercise 7: Advanced Variables, Locals, and Outputs

**Goal:** Make code reusable with inputs/outputs.

1. **Create `variables.tf`:**
   ```hcl
   variable "project_id" {
     type = string
   }

   variable "environment" {
     type        = string
     default     = "lab"
     description = "Env: lab, dev, prod"
     validation {
       condition     = contains(["lab", "dev", "prod"], var.environment)
       error_message = "Invalid environment."
     }
   }

   variable "vm_name" {
     type    = string
     default = "web-vm"
   }
   ```

2. **Update Resources:** Prefix names, e.g., VM name = "${var.environment}-${var.vm_name}".

3. **Create `locals.tf`:**
   ```hcl
   locals {
     common_labels = {
       environment = var.environment
       managed_by  = "terraform"
     }
   }
   ```
   Apply to resources: `labels = local.common_labels`.

4. **Enhanced `outputs.tf`:**
   ```hcl
   output "all_outputs" {
     value = {
       vm_ip      = google_compute_instance.web-vm.network_interface[0].access_config[0].nat_ip
       bucket     = google_storage_bucket.lab-bucket.name
       gke_host   = module.gke.endpoint
       vpc_id     = google_compute_network.vpc.id
     }
     description = "Summary of deployed resources"
   }
   ```

5. **Apply & Test:** `terraform apply -var="environment=dev"`. `terraform output all_outputs -json`.

**Best Practice:** Use `tfvars` files (e.g., `terraform.tfvars`) for env-specific values.

---

## Exercise 8: Remote State Management

**Goal:** Store state in GCS for collaboration/safety.

1. **Create State Bucket:** (One-time) In GCP Console, create `tf-state-bucket-<project-id>` (uniform access, no versioning for state).

2. **Add Backend to `main.tf`:**
   ```hcl
   terraform {
     backend "gcs" {
       bucket = "tf-state-bucket-your-project-id"
       prefix = "terraform/lab/state"
     }
   }
   ```

3. **Migrate State:** `terraform init -migrate-state`. Then `apply`.

4. **Verify:** `terraform state list` (shows all resources). In new terminal: `cd ..; git clone your-repo; cd repo; terraform init` (pulls remote state).

**Why?** Prevents local file loss; enables locking.

---

## Exercise 9: Testing, Cleanup, and Best Practices

1. **Testing:**
   - Format: `terraform fmt -recursive`.
   - Plan in CI: Use GitHub Actions with `terraform plan` on PRs.
   - Simple Test: `terraform console` > `> google_compute_network.vpc.name` (evals expression).

2. **Cleanup:** `terraform destroy --auto-approve`. Confirm: `terraform state list` (empty).

3. **Best Practices Recap:**
   | Category | Do's | Don'ts |
   |----------|------|--------|
   | **Code** | Modularize files; Use `count/for_each` for multiples. | Hardcode secrets (use Vault/Secret Manager). |
   | **State** | Remote backends; Version control `.tf` files. | Commit `tfstate` to Git. |
   | **Security** | Least privilege IAM; Encrypt buckets. | Public firewalls (`0.0.0.0/0`). |
   | **Ops** | Labels for cost tracking; Workspaces for envs (`terraform workspace new prod`). | Ignore drift—run `plan` weekly. |

4. **Extensions:**
   - Add Cloud SQL: Follow cheat-sheet example.
   - CI/CD: Integrate with Cloud Build.
   - Advanced: Use Terragrunt for DRY configs.

## Wrap-Up

Congratulations! You've deployed a full GCP stack with Terraform. Review outputs, destroy resources, and experiment. For issues, check logs with `TF_LOG=DEBUG terraform apply`.

**Resources:**
- [Terraform GCP Docs](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
- [GCP Terraform Examples](https://github.com/GoogleCloudPlatform/terraform-google-examples)
- Community: [HashiCorp Discuss](https://discuss.hashicorp.com/c/terraform-core/27)

Commit your code and share on GitHub. Questions? Ask for expansions!
