# Helm Chart

## 1. What is Helm?
- Helm helps deploy applications in Kubernetes using pre-configured templates called Helm Charts.
- A Chart is a collection of files that describe a Kubernetes application.

## Helm Basics
```bash
helm version  # Check which version of Helm is installed
helm help     # Get help with Helm commands
helm repo list # Show all added Helm repositories
```

## Adding and Updating Repositories
```bash
helm repo add <repo-name> <repo-url> # Add a repository
helm repo update                     # Get the latest list of available charts
helm search repo <chart-name>        # Search for a chart
```

## Installing Applications using Helm
```bash
helm install <release-name> <chart> # Install an application
kubectl get pods                    # See running applications in Kubernetes
```

## Listing Installed Applications
```bash
helm list # Show all installed applications
```

## Checking Application Details
```bash
helm status <release-name>     # Check the status of your installed application
helm get values <release-name> # See configuration values used
```

## Updating an Installed Application
```bash
helm upgrade <release-name> <chart> --set <parameter> # Update an installed application
```

## Uninstalling an Application
```bash
helm uninstall <release-name> # Remove an application
```

## Debugging Helm Charts
```bash
helm lint <chart>                           # Check for issues in a Helm chart
helm install --debug --dry-run <release-name> <chart> # Simulate installation
```

## Creating Your Own Helm Chart
```bash
helm create <chart-name> # Create a new Helm chart
```
