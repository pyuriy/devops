# Jenkins Comprehensive Cheatsheet

This cheatsheet provides a quick reference for Jenkins, an open-source automation server for CI/CD pipelines. It covers installation, configuration, jobs, pipelines, plugins, scheduling, best practices, and security. Based on official documentation and expert guides.

## Introduction
Jenkins enables developers to build, test, and deploy software reliably through automation. Key concepts:
- **Continuous Integration (CI)**: Frequently merge code changes into a central repository with automated builds and tests.
- **Continuous Delivery (CD)**: Automate releases to production or staging.
- **Pipelines as Code**: Define workflows in a `Jenkinsfile` using Groovy for version control and reproducibility.

## Installation
### On Ubuntu/Debian
1. Install Java:  
   ```
   sudo apt update
   sudo apt install openjdk-8-jdk  # Or openjdk-11-jdk for newer versions
   ```
2. Add Jenkins repository key:  
   ```
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   ```
3. Add repository:  
   ```
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   ```
4. Install Jenkins:  
   ```
   sudo apt update
   sudo apt install jenkins
   ```
5. Start and enable:  
   ```
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```
6. Verify:  
   ```
   sudo systemctl status jenkins
   ```
7. Access dashboard: `http://localhost:8080`. Unlock with initial admin password from `/var/lib/jenkins/secrets/initialAdminPassword`.

### Using Docker
```
docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11
```

## Starting, Stopping, and Restarting
Use systemd commands (as root or sudo):
| Action | Command |
|--------|---------|
| Start | `sudo systemctl start jenkins` |
| Stop | `sudo systemctl stop jenkins` |
| Restart | `sudo systemctl restart jenkins` |
| Status | `sudo systemctl status jenkins` |
| Enable on boot | `sudo systemctl enable jenkins` |

CLI alternative: `sudo service jenkins [start\|stop\|restart]`.

## Configuration
- **Dashboard Access**: `http://localhost:8080` (default port 8080).
- **Manage Plugins**: Dashboard > Manage Jenkins > Manage Plugins > Available tab. Filter and install (e.g., restart required for some).
- **Custom Plugin Deployment**:
  1. Stop Jenkins.
  2. Copy `.hpi` file to `$JENKINS_HOME/plugins/`.
  3. Delete expanded plugin directory.
  4. Create `<plugin>.hpi.pinned` (empty file).
  5. Start Jenkins.
- **Environment Variables**: Set in `Manage Jenkins > Configure System > Global Properties > Environment variables`.

## Job Types
Jenkins supports various project types:

| Type | Description | Use Case |
|------|-------------|----------|
| **Freestyle** | Flexible, general-purpose builds. | Simple scripts, any project. |
| **Pipeline** | Workflow as code in Jenkinsfile. | Full CI/CD pipelines. |
| **Multiconfiguration** | Same job on multiple environments. | Cross-platform testing. |
| **Folder** | Organizes jobs in hierarchies. | Grouping related projects. |
| **GitHub Organization** | Auto-creates pipelines from GitHub repos with Jenkinsfile. | GitHub-centric teams. |
| **Multibranch Pipeline** | Branch-specific Jenkinsfiles. | Monorepos with branches. |

### Creating a Job
1. Dashboard > New Item.
2. Enter name, select type (e.g., Pipeline).
3. Configure: SCM, build steps, post-build actions.
4. Save and run: Build Now.

## Build Pipelines (Freestyle Chaining)
To chain jobs (e.g., Job1 → Job2 → Job3):
1. Install Build Pipeline Plugin (Manage Plugins > Available).
2. For each job: Configure > Post-build Actions > Build other projects (add downstream job).
3. Create view: Dashboard > + (New View) > Build Pipeline View > Configure > Select initial job.

## Jenkins Pipelines
Two syntaxes: **Declarative** (structured, recommended) and **Scripted** (flexible Groovy).

### Creating a Pipeline Job
1. New Item > Pipeline.
2. Pipeline Definition: 
   - Pipeline script (inline Groovy).
   - Pipeline script from SCM (e.g., Git URL, Jenkinsfile path).
3. Save.

### Declarative Pipeline Structure
```
pipeline {
    agent any  // Or specific label/docker
    environment { KEY = 'value' }
    parameters { string(name: 'PARAM', defaultValue: 'default') }
    options { timeout(time: 1, unit: 'HOURS') }
    triggers { cron('H/15 * * * *') }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('Test') {
            parallel {
                stage('Unit') { steps { sh 'mvn test' } }
                stage('Integration') { steps { sh 'mvn verify' } }
            }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps { sh 'deploy.sh' }
        }
    }
    post {
        always { cleanWs() }
        success { emailext(...) }
        failure { slackSend(...) }
    }
}
```

### Scripted Pipeline Structure
```
node('label') {
    stage('Build') {
        // Groovy code
        sh 'mvn clean compile'
    }
    try {
        stage('Test') { sh 'mvn test' }
    } catch (e) {
        echo 'Test failed!'
        throw e
    }
}
```

### Key Directives and Steps
- **Agent**: `agent any`, `agent { docker 'image' }`, `agent { label 'linux' }`.
- **Stages/Steps**: Sequential execution; use `parallel { ... }` for concurrency.
- **Environment**: `environment { CC = 'clang' }`; credentials: `credentials('id')`.
- **Parameters**: `string`, `choice`, `booleanParam`, `password`. Access via `params.NAME`.
- **Triggers**: `cron('H * * * *')`, `pollSCM('H/5 * * * *')`.
- **Options**: `timeout`, `retry(3)`, `timestamps`, `buildDiscarder(logRotator(numToKeepStr: '5'))`.
- **When**: Conditions like `when { branch 'main' }` or `allOf { branch 'main'; expression { ... } }`.
- **Input**: `input { message 'Approve?' }`.
- **Matrix**: For combinatorial builds: `matrix { axes { axis { name 'OS' values 'linux', 'windows' } } }`.
- **Post**: Conditions: `always`, `success`, `failure`, `cleanup`.

### Snippet Generator
In Pipeline job config > Pipeline Syntax > Select step (e.g., Git) > Generate > Copy code.

## Plugins
Over 1,800 available. Install via Manage Plugins.

| Plugin | Purpose |
|--------|---------|
| Git | SCM integration. |
| Maven Integration | Build automation. |
| Pipeline | Declarative/scripted pipelines. |
| Docker Pipeline | Docker builds. |
| Blue Ocean | Modern UI for pipelines. |
| Slack Notification | Alerts. |
| Email Extension | Email reports. |
| Copy Artifact | Share builds between jobs. |
| HTML Publisher | Publish reports. |
| Amazon EC2 | Dynamic agents. |

**Best Practice**: Vet plugins for security; update regularly.

## Build Triggers and Scheduling (Cron)
Use 5-field cron: `MINUTE HOUR DOM MONTH DOW` (0-59, 0-23, 1-31, 1-12, 0-7; 0/7=Sunday). Use `H` for hash-based load distribution.

| Schedule | Cron Expression | Description |
|----------|-----------------|-------------|
| Every minute | `* * * * *` | Run continuously. |
| Every 15 min | `H/15 * * * *` | Balanced every 15 min. |
| Every hour | `H * * * *` | Hourly. |
| Weekdays 9 AM | `H 9 * * 1-5` | Business hours. |
| 2:30 PM on 1st/15th | `H 14 1,15 * *` | Semi-monthly. |
| Poll SCM every 5 min | `pollSCM('H/5 * * * *')` | Check for changes. |

In job config: Build Triggers > Build periodically or Poll SCM.

## Best Practices
- **Version Control Pipelines**: Store Jenkinsfile in SCM.
- **Isolation**: Use ephemeral agents (e.g., Docker) per build.
- **Testing**: Include unit/integration/UAT in pipelines.
- **Monitoring**: Enable timestamps; log failures.
- **Scalability**: Use labels for agent selection; dynamic clouds (e.g., EC2 plugin).
- **Error Handling**: Use `try-catch` in scripted; `post { failure { ... } }` in declarative.
- **Security Scans**: Integrate SAST/DAST in pipeline.
- **Approval Gates**: Use `input` for production deploys.

## Security
Secure Jenkins to prevent common risks like exposed instances, credential leaks, and plugin exploits.

### Enabling and Basic Setup
- **Enable Security**: Manage Jenkins > Configure Global Security > Enable Security.
- **Authorization**: Use Matrix-based or Role-based strategy (install Role-based Authorization Strategy plugin).
- **Authentication**: LDAP, SAML, or Jenkins' own DB. Enforce MFA where possible.
- **Least Privilege**: Deny by default; assign minimal roles (e.g., read-only for most users).

### Access Control
- Restrict dashboard access: IP whitelisting via proxy (e.g., Nginx).
- Controller Isolation: Run builds on separate, ephemeral nodes.
- Disable unused features: Anonymous access, CLI over HTTP.

### Secrets Management
- **Never Hardcode**: Use Credentials Store (Manage Jenkins > Manage Credentials).
- **Tools**: Integrate HashiCorp Vault, AWS Secrets Manager.
- **Best Practices**:
  - Encrypt at rest; no plaintext in logs/console.
  - Temporary creds/OTPs; rotate regularly.
  - Scan for leaks: Use git-secrets in pipelines.
  - Access: `withCredentials([usernamePassword(credentialsId: 'id', usernameVariable: 'USER', passwordVariable: 'PASS')]) { sh 'use $USER $PASS' }`.

### Third-Party and Plugins
- Vet plugins: Check popularity, updates, security history.
- Limit installs: Only admins can add; review changes.
- Update promptly; remove unused.

### Pipeline Security
- **Integrity**: Require signed commits; verify package checksums.
- **Isolation**: Avoid `--privileged` in Docker; use non-root users.
- **Scans**: Embed SAST (e.g., SonarQube), IaC scanning.
- **Approvals**: Manual gates for prod deploys.
- **Version Control Config**: Store pipelines outside code repo if sensitive.

### Visibility and Monitoring
- **Logging**: Enable audit logs (Audit Trail plugin); avoid sensitive data.
- **Alerts**: Integrate with SIEM; monitor for anomalies (e.g., failed logins).
- **Inventory**: Track identities, permissions, last-used creds.

For full details, refer to official docs and OWASP guidelines.
