# Troubleshooting Guide - Terraform CDP Configuration

**Date:** November 25, 2025  
**Project:** CDP Test Environment on GCP  
**User:** yuriy.petyuk@139canadainc.com

---

## Issues Encountered and Resolutions

### 1. GCloud Organization Policy Error - Project Configuration

#### Issue
```bash
ERROR: (gcloud.org-policies.delete) The value of ``core/project'' property is set to project number.
To use this command, set ``--project'' flag to PROJECT ID or set ``core/project'' property to PROJECT ID.
```

#### Root Cause
The `gcloud` configuration had `core/project` set to a project **number** instead of a project **ID**, which caused issues when running organization-level commands.

#### Resolution Options

**Option 1: Temporarily unset the project property**
```bash
gcloud config unset project
gcloud org-policies delete <POLICY_NAME> --organization=<ORG_ID>
```

**Option 2: Set a valid project ID**
```bash
gcloud config set project YOUR_PROJECT_ID
```

**Option 3: Check and convert project number to ID**
```bash
gcloud config get-value project
gcloud projects list --filter="projectNumber:YOUR_PROJECT_NUMBER"
gcloud config set project YOUR_PROJECT_ID
```

---

### 2. GCloud Organization Policy - Permission Denied

#### Issue
```bash
ERROR: (gcloud.org-policies.delete) PERMISSION_DENIED: Permission 'orgpolicy.policies.delete' 
denied on resource '//orgpolicy.googleapis.com/organizations/799194967374/policies/
iam.managed.disableServiceAccountApiKeyCreation'
```

#### Root Cause
User account lacked the `orgpolicy.policies.delete` permission at the organization level.

#### Resolution
Contact organization administrator to grant one of these IAM roles:
- **Organization Policy Administrator** (`roles/orgpolicy.policyAdmin`)
- **Organization Administrator** (`roles/resourcemanager.organizationAdmin`)

```bash
# Organization admin would run:
gcloud organizations add-iam-policy-binding 799194967374 \
  --member="user:yuriy.petyuk@139canadainc.com" \
  --role="roles/orgpolicy.policyAdmin"
```

---

### 3. GCloud Organization Policy - Incorrect Policy Name

#### Issue
```bash
ERROR: (gcloud.org-policies.delete) NOT_FOUND: Requested entity was not found.
```

#### Root Cause
Attempted to delete policy with incorrect constraint name:
- ❌ Used: `iam.managed.disableServiceAccountApiKeyCreation`
- ✅ Actual: `iam.disableServiceAccountKeyCreation`

#### Resolution
List existing organization policies to verify correct names:
```bash
gcloud org-policies list --organization=799194967374
```

**Current Organization Policies:**
```
CONSTRAINT                                          LIST_POLICY  BOOLEAN_POLICY
iam.allowedPolicyMemberDomains                      SET          -
iam.disableServiceAccountKeyCreation                -            SET
storage.uniformBucketLevelAccess                    -            SET
essentialcontacts.allowedContactDomains             SET          -
compute.restrictProtocolForwardingCreationForTypes  SET          -
iam.automaticIamGrantsForDefaultServiceAccounts     -            SET
iam.disableServiceAccountKeyUpload                  -            SET
compute.setNewProjectDefaultToZonalDNSOnly          -            SET
```

**Correct command:**
```bash
gcloud org-policies delete iam.disableServiceAccountKeyCreation --organization=799194967374
```

---

### 4. Terraform Version Incompatibility

#### Issue
```bash
Error: Invalid reference in variable validation
Error: Invalid validation error message
Error: Unsupported argument: "sensitive" is not expected here
Error: Invalid type specification: Keyword "optional" is not a valid type constructor
```

#### Root Cause
- Running **Terraform v0.13.0** (released 2020)
- Cloudera CDP modules require **Terraform v1.3+** for:
  - `optional()` type constructor (added in v1.3)
  - `sensitive` attribute on variables (added in v0.14)
  - Enhanced validation features

#### Resolution
Upgrade Terraform to the latest version (v1.14.0):

**Step 1: Download Terraform binary**
```bash
wget https://releases.hashicorp.com/terraform/1.14.0/terraform_1.14.0_linux_amd64.zip
```

**Step 2: Extract and install**
```bash
unzip terraform_1.14.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```

**Step 3: Verify installation**
```bash
terraform version
# Output: Terraform v1.14.0
```

**Step 4: Clean up**
```bash
rm terraform_1.14.0_linux_amd64.zip LICENSE.txt
```

---

### 5. Google Provider Version Conflict

#### Issue
```bash
Error: Failed to query available provider packages
Could not retrieve the list of available versions for provider hashicorp/google: 
no available releases match the given constraints ~> 4.0, >= 6.12.0
```

#### Root Cause
Conflicting provider version requirements:
- `main.tf` specified: `~> 4.0` (versions 4.0.x - 4.99.x)
- GCP prereqs module required: `>= 6.12.0` (version 6.12.0 or higher)

No version satisfies both constraints.

#### Resolution
Update the Google provider version constraint in `main.tf`:

**Before:**
```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"  # ❌ Too old
    }
    cloudera = {
      source  = "cloudera/cdp"
      version = "~> 0.2"
    }
  }
}
```

**After:**
```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = ">= 6.12.0"  # ✅ Compatible
    }
    cloudera = {
      source  = "cloudera/cdp"
      version = "~> 0.2"
    }
  }
}
```

**Reinitialize Terraform:**
```bash
terraform init -upgrade
```

**Result:**
- Installed: `google` provider v7.12.0
- All module dependencies satisfied

---

## Final Validation

After all fixes were applied:

```bash
$ terraform validate
Success! The configuration is valid.
```

---

## System Information

**Operating System:**
```
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.6 LTS
Release:        20.04
Codename:       focal
```

**Terraform Versions:**
- Before: v0.13.0
- After: v1.14.0

**Google Cloud Provider:**
- Before: v4.x (constraint)
- After: v7.12.0

**GCP Organization ID:** 799194967374  
**GCP Project:** terraform-479301  
**GCP Region:** northamerica-northeast2

---

## Key Takeaways

1. **Always check organization policy names** - Use `gcloud org-policies list` to verify exact constraint names
2. **Keep Terraform up to date** - Modern modules require recent Terraform versions for advanced features
3. **Resolve provider version conflicts** - Ensure all modules can agree on compatible provider versions
4. **Verify permissions** - Organization-level operations require appropriate IAM roles
5. **Project ID vs Number** - GCloud commands often require project ID, not project number

---

## Useful Commands Reference

### GCloud
```bash
# List organization policies
gcloud org-policies list --organization=<ORG_ID>

# Check current project configuration
gcloud config get-value project

# List projects
gcloud projects list

# Reset organization policy (less permissions required than delete)
gcloud org-policies reset <CONSTRAINT_NAME> --organization=<ORG_ID>
```

### Terraform
```bash
# Check version
terraform version

# Validate configuration
terraform validate

# Check provider requirements
terraform providers

# Initialize with upgrades
terraform init -upgrade

# Preview changes
terraform plan

# Apply changes
terraform apply
```

---

## Additional Resources

- [Terraform Downloads](https://www.terraform.io/downloads.html)
- [Google Cloud Organization Policies](https://cloud.google.com/resource-manager/docs/organization-policy/overview)
- [Cloudera CDP Terraform Modules](https://github.com/cloudera-labs/terraform-cdp-modules)
- [Terraform Google Provider Documentation](https://registry.terraform.io/providers/hashicorp/google/latest/docs)

---

**Document Status:** ✅ All issues resolved  
**Configuration Status:** ✅ Validated and ready for deployment
