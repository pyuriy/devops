Below is a **comprehensive lab guide** for setting up, configuring, and using Jenkins to build a CI/CD pipeline. This lab assumes you're starting from scratch and will guide you through installing Jenkins, setting up a simple project, creating a pipeline, integrating with Git, running builds, and exploring security and plugins. The lab is designed to be hands-on, practical, and suitable for beginners to intermediate users.

---

# Jenkins Comprehensive Lab Guide

## Objective
By the end of this lab, you will:
1. Install and configure Jenkins on a local machine or cloud instance.
2. Create a Freestyle job and a Pipeline job.
3. Integrate Jenkins with Git for source control.
4. Build and test a sample application.
5. Configure basic security and install plugins.
6. Explore pipeline syntax and triggers.

## Prerequisites
- **System**: Ubuntu 20.04+ (or Docker-capable OS) or a cloud VM (e.g., AWS EC2, GCP).
- **Software**: Git, Java 8/11, Docker (optional), internet access.
- **Accounts**: GitHub account for SCM integration.
- **Basic Knowledge**: Familiarity with terminal commands, Git, and basic programming.

## Lab Setup
### Option 1: Install Jenkins on Ubuntu
1. **Update System**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. **Install Java**:
   ```bash
   sudo apt install openjdk-11-jdk -y
   java -version  # Verify: openjdk 11.x
   ```
3. **Add Jenkins Repository**:
   ```bash
   wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   ```
4. **Install Jenkins**:
   ```bash
   sudo apt update
   sudo apt install jenkins -y
   ```
5. **Start Jenkins**:
   ```bash
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   sudo systemctl status jenkins  # Ensure it's running
   ```
6. **Access Jenkins**:
   - Open browser: `http://<your-server-ip>:8080` (or `http://localhost:8080` if local).
   - Retrieve initial admin password:
     ```bash
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
   - Copy password, unlock Jenkins, and proceed.

### Option 2: Run Jenkins with Docker
1. **Install Docker** (if not installed):
   ```bash
   sudo apt install docker.io -y
   sudo systemctl start docker
   sudo systemctl enable docker
   ```
2. **Run Jenkins Container**:
   ```bash
   docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home --name jenkins jenkins/jenkins:lts-jdk11
   ```
3. **Get Admin Password**:
   ```bash
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```
4. **Access Jenkins**: `http://localhost:8080`.

### Initial Jenkins Setup
1. **Complete Setup Wizard**:
   - Paste admin password.
   - Install suggested plugins (takes ~5-10 min).
   - Create admin user (e.g., username: `admin`, password: `admin123`).
   - Set instance URL (e.g., `http://localhost:8080`).
2. **Verify Dashboard**: Ensure you see the Jenkins dashboard.

## Lab 1: Create a Freestyle Project
### Goal
Build a simple job to run a shell script and verify Jenkins functionality.

1. **Create Project**:
   - Dashboard > New Item.
   - Name: `HelloWorldJob`.
   - Type: Freestyle project > OK.
2. **Configure Job**:
   - General: Add description, e.g., "Simple shell job".
   - Build Steps: Add build step > Execute shell.
   - Script:
     ```bash
     echo "Hello, Jenkins!"
     date
     ```
3. **Save and Run**:
   - Save > Build Now.
   - Check Build History (#1) > Console Output. Look for `Hello, Jenkins!` and timestamp.
4. **Test Failure**:
   - Edit job > Build Steps > Add:
     ```bash
     exit 1  # Simulates failure
     ```
   - Save > Build Now. Verify failure in Console Output (red icon in Build History).

## Lab 2: Set Up Git Integration
### Goal
Connect Jenkins to a GitHub repository and pull code.

1. **Create a Sample Repo**:
   - On GitHub, create a public repo (e.g., `jenkins-lab`).
   - Add a file `hello.sh`:
     ```bash
     #!/bin/bash
     echo "Hello from GitHub!"
     ```
   - Commit and push.
2. **Install Git Plugin** (if not already installed):
   - Manage Jenkins > Manage Plugins > Available > Search "Git" > Install without restart.
3. **Create Freestyle Job**:
   - New Item > Name: `GitJob` > Freestyle project.
   - Source Code Management: Select Git.
   - Repository URL: `https://github.com/<your-username>/jenkins-lab.git` (public repo, no credentials needed).
   - Branch: `*/main`.
   - Build Steps > Execute shell:
     ```bash
     chmod +x hello.sh
     ./hello.sh
     ```
   - Save > Build Now.
4. **Verify**:
   - Console Output should show Git clone and `Hello from GitHub!`.

## Lab 3: Create a Declarative Pipeline
### Goal
Define a pipeline using a `Jenkinsfile` for a multi-stage CI process.

1. **Update GitHub Repo**:
   - In `jenkins-lab` repo, create `Jenkinsfile`:
     ```groovy
     pipeline {
         agent any
         stages {
             stage('Clone') {
                 steps {
                     git url: 'https://github.com/<your-username>/jenkins-lab.git', branch: 'main'
                 }
             }
             stage('Build') {
                 steps {
                     sh 'echo Building...'
                     sh 'chmod +x hello.sh'
                     sh './hello.sh'
                 }
             }
             stage('Test') {
                 steps {
                     sh 'echo Running tests...'
                     sh 'test -f hello.sh && echo "File exists" || exit 1'
                 }
             }
             stage('Deploy') {
                 steps {
                     sh 'echo Deploying to production...'
                 }
             }
         }
         post {
             success {
                 echo 'Pipeline completed successfully!'
             }
             failure {
                 echo 'Pipeline failed!'
             }
         }
     }
     ```
   - Commit and push.
2. **Create Pipeline Job**:
   - New Item > Name: `MyPipeline` > Pipeline.
   - Pipeline > Definition: Pipeline script from SCM.
   - SCM: Git.
   - Repository URL: `https://github.com/<your-username>/jenkins-lab.git`.
   - Branch: `*/main`.
   - Script Path: `Jenkinsfile`.
   - Save > Build Now.
3. **Verify**:
   - Check Stage View (requires Pipeline plugin).
   - Console Output: Look for `Clone`, `Build`, `Test`, `Deploy`, and `Pipeline completed successfully!`.
4. **Test Failure**:
   - Edit `Jenkinsfile`, change `Test` stage to:
     ```groovy
     sh 'exit 1'  # Force failure
     ```
   - Push to GitHub > Build Now. Verify `Pipeline failed!` in Console Output.

## Lab 4: Scheduling and Triggers
### Goal
Automate builds using cron and SCM polling.

1. **Modify Pipeline Job**:
   - Edit `MyPipeline` > Build Triggers.
   - Check Poll SCM > Schedule: `H/5 * * * *` (every 5 minutes).
2. **Test Trigger**:
   - Update `hello.sh` in GitHub (e.g., change echo message).
   - Commit and push.
   - Wait ~5 minutes; check Build History for new build.
3. **Add Periodic Trigger**:
   - Edit job > Build Triggers > Check Build periodically.
   - Schedule: `H * * * *` (hourly).
   - Save. Verify builds occur hourly.

## Lab 5: Install and Use Plugins
### Goal
Enhance Jenkins with plugins for notifications and reporting.

1. **Install Email Extension Plugin**:
   - Manage Jenkins > Manage Plugins > Available > Search "Email Extension" > Install.
2. **Configure Email**:
   - Manage Jenkins > Configure System > Extended E-mail Notification.
   - SMTP Server: Use a test server (e.g., `smtp.gmail.com`, port 587).
   - Credentials: Add SMTP credentials (for Gmail, use App Password).
   - Test: Send test email to your address.
3. **Add Email to Pipeline**:
   - Edit `Jenkinsfile`, add to `post` section:
     ```groovy
     post {
         success {
             emailext subject: 'Build Success', body: 'Pipeline completed!', to: 'your.email@example.com'
         }
         failure {
             emailext subject: 'Build Failed', body: 'Check logs!', to: 'your.email@example.com'
         }
     }
     ```
   - Push changes > Build Now. Check email for notification.
4. **Explore Blue Ocean**:
   - Install Blue Ocean plugin.
   - Access: Dashboard > Blue Ocean.
   - View `MyPipeline` in visual format.

## Lab 6: Basic Security Configuration
### Goal
Secure Jenkins to prevent unauthorized access.

1. **Enable Security**:
   - Manage Jenkins > Configure Global Security.
   - Check Enable Security.
   - Authentication: Jenkins’ own user database.
   - Authorization: Matrix-based security.
2. **Create User**:
   - Manage Jenkins > Manage Users > Create User (e.g., `dev`, password: `dev123`).
3. **Set Permissions**:
   - In Matrix-based security, add `dev`.
   - Grant: Read, Job > Read, View > Read (uncheck others).
   - Save. Log out and test `dev` login (limited access).
4. **Disable Anonymous Access**:
   - In Configure Global Security, uncheck Anonymous > Read.
   - Save. Verify anonymous users can’t access dashboard.

## Lab 7: Build a Sample Application
### Goal
Build and test a simple Java application.

1. **Create Java Project**:
   - In `jenkins-lab` repo, add `HelloWorld.java`:
     ```java
     public class HelloWorld {
         public static void main(String[] args) {
             System.out.println("Hello, Jenkins World!");
         }
     }
     ```
   - Add `pom.xml` for Maven:
     ```xml
     <project>
         <modelVersion>4.0.0</modelVersion>
         <groupId>com.example</groupId>
         <artifactId>jenkins-lab</artifactId>
         <version>1.0-SNAPSHOT</version>
         <build>
             <plugins>
                 <plugin>
                     <groupId>org.apache.maven.plugins</groupId>
                     <artifactId>maven-compiler-plugin</artifactId>
                     <version>3.8.1</version>
                     <configuration>
                         <source>11</source>
                         <target>11</target>
                     </configuration>
                 </plugin>
             </plugins>
         </build>
     </project>
     ```
   - Push to GitHub.
2. **Install Maven**:
   - On Jenkins server:
     ```bash
     sudo apt install maven -y
     mvn -version
     ```
3. **Update Jenkinsfile**:
   ```groovy
   pipeline {
       agent any
       tools {
           maven 'Maven'  # Ensure Maven is configured in Global Tool Configuration
       }
       stages {
           stage('Clone') {
               steps {
                   git url: 'https://github.com/<your-username>/jenkins-lab.git', branch: 'main'
               }
           }
           stage('Build') {
               steps {
                   sh 'mvn clean compile'
               }
           }
           stage('Test') {
               steps {
                   sh 'mvn test'  # Add tests later if needed
               }
           }
           stage('Run') {
               steps {
                   sh 'java -cp target/classes HelloWorld'
               }
           }
       }
   }
   ```
4. **Configure Maven in Jenkins**:
   - Manage Jenkins > Global Tool Configuration > Maven > Add Maven (name: `Maven`, auto-install).
   - Save.
5. **Build and Verify**:
   - Build `MyPipeline`. Check Console Output for `Hello, Jenkins World!`.

## Cleanup
1. **Stop Jenkins**:
   - Ubuntu: `sudo systemctl stop jenkins`.
   - Docker: `docker stop jenkins`.
2. **Remove Docker Data** (if used):
   ```bash
   docker rm jenkins
   docker volume rm jenkins_home
   ```
3. **Delete GitHub Repo** (optional).

## Optional Challenges
1. **Add Unit Tests**: Write JUnit tests for `HelloWorld.java` and integrate in `Test` stage.
2. **Docker Pipeline**: Modify pipeline to build a Docker image.
3. **Slack Notifications**: Install Slack Notification plugin and send build alerts.
4. **Parameterized Builds**: Add `parameters` to `Jenkinsfile` (e.g., choose branch).

## Troubleshooting
- **Port 8080 Blocked**: Check firewall (`sudo ufw allow 8080`) or change port in `/etc/default/jenkins` (HTTP_PORT).
- **Git Errors**: Verify repo URL; for private repos, add credentials in Jenkins (Manage Jenkins > Manage Credentials).
- **Plugin Issues**: Check compatibility; restart Jenkins after installs (`sudo systemctl restart jenkins`).
- **Pipeline Syntax Errors**: Use Pipeline Syntax generator (in job config > Pipeline Syntax).

## Resources
- Official Docs: `https://www.jenkins.io/doc/`
- Pipeline Syntax: `http://<your-jenkins>/pipeline-syntax/`
- Plugin Index: `https://plugins.jenkins.io/`
- GitHub: `https://github.com/jenkinsci/`

This lab provides a hands-on introduction to Jenkins’ core features. Extend it by exploring advanced plugins, cloud agents, or complex pipelines based on your project needs.
