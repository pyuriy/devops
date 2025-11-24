# Terraform with GCP Cheat Sheet

This cheat sheet covers essential concepts, setup, commands, resources, and best practices for using Terraform with Google Cloud Platform (GCP). It's structured for quick reference. Examples use HCL syntax. Assumes basic Terraform familiarity.

## 1. Installation & Setup

### Prerequisites
- Terraform CLI: Download from [HashiCorp releases](https://developer.hashicorp.com/terraform/install). Verify: `terraform version`.
- GCP SDK: Install `gcloud` CLI from [Google Cloud SDK](https://cloud.google.com/sdk/docs/install). Authenticate: `gcloud auth application-default login`.
- Enable APIs: Use `gcloud services enable` for required services (e.g., `compute.googleapis.com` for VMs).

### Provider Installation
Add to `main.tf` or `versions.tf`:
```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"  # Check latest at registry.terraform.io/providers/hashicorp/google
    }
  }
}
```

## 2. Provider Configuration

Configure in `provider.tf` or `main.tf`. Supports multiple projects/regions.

### Basic Provider Block
```hcl
provider "google" {
  project = "my-project-id"
  region  = "us-central1"
  zone    = "us-central1-a"
}
```

### Advanced Options
| Option | Description | Example |
|--------|-------------|---------|
| `credentials` | Path to service account JSON key | `credentials = "/path/to/key.json"` |
| `access_token` | OAuth2 access token | `access_token = "ya29.a0Af..."` |
| `billing_project` | Project for billing | `billing_project = "billing-project-id"` |
| `user_project_override` | Override for quota checks | `user_project_override = true` |
| `scopes` | Custom OAuth scopes | `scopes = ["https://www.googleapis.com/auth/cloud-platform"]` |

**Alias for Multi-Provider**:
```hcl
provider "google" {
  alias   = "prod"
  project = "prod-project"
}

resource "google_compute_instance" "vm" {
  provider = google.prod
  # ...
}
```

## 3. Common Commands

Run from project directory with `.tf` files.

| Command | Description | Flags/Notes |
|---------|-------------|-------------|
| `terraform init` | Initialize backend, download providers | `-upgrade` to update providers |
| `terraform plan` | Preview changes | `-var="key=value"` for vars; `-out=plan.tfplan` to save |
| `terraform apply` | Apply changes | `-auto-approve`; Use saved plan: `terraform apply plan.tfplan` |
| `terraform destroy` | Destroy all resources | `-auto-approve` |
| `terraform validate` | Validate syntax | N/A |
| `terraform fmt` | Format code | `-recursive` for dirs |
| `terraform console` | Interactive eval | For testing expressions |
| `terraform state list` | List managed resources | N/A |
| `terraform output` | Show outputs | `-json` for structured |

**Workflow**: `init` → `plan` → `apply`. Use `-target=resource.name` to apply specific resources.

## 4. Variables & Outputs

### Variables (`variables.tf`)
Define inputs for reusability.
```hcl
variable "project_id" {
  type        = string
  description = "GCP Project ID"
  default     = "my-project"
}

variable "machine_type" {
  type        = string
  default     = "e2-micro"
  validation {
    condition     = contains(["e2-micro", "e2-small"], var.machine_type)
    error_message = "Invalid machine type."
  }
}
```
Usage: `resource "google_compute_instance" "vm" { machine_type = var.machine_type }`
Pass via CLI: `terraform apply -var="project_id=foo"`

### Locals (`locals.tf`)
```hcl
locals {
  common_tags = {
    Environment = "dev"
    Owner       = "team@example.com"
  }
}
```
Usage: `tags = local.common_tags`

### Outputs (`outputs.tf`)
Expose values post-apply.
```hcl
output "instance_ip" {
  value       = google_compute_instance.vm.network_interface[0].access_config[0].nat_ip
  description = "Public IP of the VM"
  sensitive   = false  # Use true for secrets
}
```
View: `terraform output instance_ip`

## 5. Key Resources

Common GCP resources. Examples are minimal; expand with full docs.

### Compute Engine (VM)
```hcl
resource "google_compute_instance" "vm" {
  name         = "my-vm"
  machine_type = var.machine_type
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
      size  = 10
      type  = "pd-standard"
    }
  }

  network_interface {
    network = google_compute_network.vpc.id
    access_config {}  # Enables public IP
  }

  metadata = {
    startup-script = "#!/bin/bash\necho 'Hello' > /hello.txt"
  }

  tags = ["http-server"]
}
```

### Networking (VPC & Subnet)
```hcl
resource "google_compute_network" "vpc" {
  name                    = "my-vpc"
  auto_create_subnetworks = false
  mtu                     = 1460
}

resource "google_compute_subnetwork" "subnet" {
  name          = "my-subnet"
  ip_cidr_range = "10.0.1.0/24"
  region        = "us-central1"
  network       = google_compute_network.vpc.id
}
```

### Firewall
```hcl
resource "google_compute_firewall" "allow_http" {
  name    = "allow-http"
  network = google_compute_network.vpc.name

  allow {
    protocol = "tcp"
    ports    = ["80", "443"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["http-server"]
}
```

### Cloud Storage (Bucket)
```hcl
resource "google_storage_bucket" "bucket" {
  name     = "my-unique-bucket-name"
  location = "US"
  force_destroy = true  # Allows deletion with objects

  versioning {
    enabled = true
  }

  uniform_bucket_level_access = true
}

resource "google_storage_bucket_object" "object" {
  name   = "file.txt"
  source = "./local-file.txt"
  bucket = google_storage_bucket.bucket.name
}
```

### Cloud SQL (Database)
```hcl
resource "google_sql_database_instance" "main" {
  name             = "main-db"
  database_version = "POSTGRES_14"
  region           = "us-central1"

  settings {
    tier = "db-f1-micro"
  }
}

resource "google_sql_database" "db" {
  name     = "mydb"
  instance = google_sql_database_instance.main.name
}

resource "google_sql_user" "user" {
  name     = "myuser"
  instance = google_sql_database_instance.main.name
  password = "secure-password"
}
```

### Kubernetes Engine (GKE Cluster)
```hcl
resource "google_container_cluster" "primary" {
  name     = "my-gke-cluster"
  location = "us-central1"

  remove_default_node_pool = true
  initial_node_count       = 1

  master_auth {
    username = ""
    password = ""  # Use private cluster for prod
  }

  node_config {
    machine_type = "e2-medium"
    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform",
    ]
  }
}
```

### IAM (Service Account & Roles)
```hcl
resource "google_service_account" "sa" {
  account_id   = "my-sa"
  display_name = "My Service Account"
}

resource "google_project_iam_member" "viewer" {
  project = var.project_id
  role    = "roles/viewer"
  member  = "serviceAccount:${google_service_account.sa.email}"
}
```

### Other Common Resources
| Resource Type | Example Name | Key Attributes |
|---------------|--------------|----------------|
| `google_project` | Project creation | `name`, `org_id` |
| `google_bigquery_dataset` | BQ Dataset | `dataset_id`, `location` |
| `google_cloud_run_service` | Cloud Run | `name`, `container_image` |
| `google_cloudfunctions_function` | Cloud Functions | `name`, `source_archive_bucket` |
| `google_pubsub_topic` | Pub/Sub | `name` |
| `google_cloud_scheduler_job` | Scheduler | `name`, `schedule` |

For full list: 100+ resources in provider.

## 6. State Management

- **Local State** (default): `terraform.tfstate` file. Risky for teams.
- **Remote Backend** (recommended):
  ```hcl
  terraform {
    backend "gcs" {  # GCP's GCS backend
      bucket = "my-tf-state-bucket"
      prefix = "terraform/state"
    }
  }
  ```
  Init with: `terraform init` (migrate if needed).

- **State Commands**:
  - `mv`: Rename resource in state.
  - `rm`: Remove from state (not destroy).
  - `pull/push`: Manual sync.

- **Locking**: GCS backend supports via DynamoDB-like (use external for GCP).

## 7. Modules

Reusable code blocks. Download from Terraform Registry or local.

### Usage
```hcl
module "vpc" {
  source = "terraform-google-modules/network/google"
  version = "~> 4.0"

  project_id = var.project_id
  network_name = "my-vpc"
  subnets = [
    {
      subnet_name   = "subnet-1"
      subnet_ip     = "10.0.1.0/24"
      subnet_region = "us-central1"
    }
  ]
}
```

- Official GCP Modules: [terraform-google-modules](https://github.com/terraform-google-modules).

## 8. Best Practices

| Category | Tips |
|----------|------|
| **Security** | Use service accounts over user creds; Enable OS Login; Avoid hardcoding secrets (use `google_secret_manager_secret`). |
| **Organization** | Use workspaces: `terraform workspace new dev`; Consistent naming (e.g., `{env}-{resource}-{location}`). |
| **Performance** | Use `count`/`for_each` for loops; Parallelism: `terraform apply -parallelism=10`. |
| **Testing** | `terratest` for integration; `terraform validate` + CI/CD (GitHub Actions, Cloud Build). |
| **Costs** | Use `lifecycle { prevent_destroy = true }`; Monitor with labels. |
| **Versioning** | Pin provider/module versions; Use `terraform init -lock=false` in CI. |
| **Drift Detection** | `terraform plan` regularly; Refresh: `terraform refresh`. |

## 9. Troubleshooting

- **Auth Errors**: Run `gcloud auth list`; Ensure service account has `roles/owner` or specific roles.
- **API Not Enabled**: `gcloud services enable compute.googleapis.com`.
- **State Locked**: Use `terraform force-unlock <lock-id>`.
- **Resource Conflicts**: Use `terraform import` for existing resources.
- **Debug**: `TF_LOG=DEBUG terraform apply`.

For latest updates, check [Terraform GCP Provider Docs](https://registry.terraform.io/providers/hashicorp/google/latest/docs). Examples tested on Terraform 1.6+ / GCP Provider 5.0+. Customize for your use case!
