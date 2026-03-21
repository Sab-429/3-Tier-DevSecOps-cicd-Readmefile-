COMPLETE DEVSECOPS 3-TIER CI/CD PIPELINE (AWS + JENKINS)

---

☁️ AWS EC2 SETUP

Launch EC2:
- Ubuntu 22.04
- minimum 4GB memory machine
- 12GB storage

Security Group Inbound Rules:
- 22   → SSH
- 8080 → Jenkins
- 9000 → SonarQube
- 80   → Application
- 443  → HTTPS (optional)

---


## 🔧 SYSTEM SETUP

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🐳 DOCKER INSTALLATION

```bash
sudo apt install -y docker.io
sudo apt install -y docker-compose
```
```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```
```bash
sudo reboot
```

Verify:

```bash
docker --version
```

--- 

## JENKINS INSTALLATION

Update the Debian apt repositories, install OpenJDK 21, and check the installation using the following commands:

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
```

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```
```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
Check:
```bash
sudo systemctl status jenkins
```

---


## ACCESS JENKINS

http://<EC2-IP>:8080

Get password:
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

- Install suggested plugins
- Create admin user

---


## INSTALL JENKINS PLUGINS

Manage Jenkins → Plugins → Install:

- Git
- Pipeline
- SonarQube Scanner
- Docker Pipeline
- OWASP Dependency-Check

Restart Jenkins after install.

========================
🔍 SONARQUBE SETUP (DOCKER)
========================

docker pull sonarqube:lts

docker run -d \
--name sonarqube \
-p 9000:9000 \
sonarqube:lts

Access:
http://<EC2-IP>:9000

Login:
admin / admin

Generate Token:
My Account → Security → Generate Token

========================
🔐 CONNECT SONARQUBE WITH JENKINS
========================

Manage Jenkins → System → SonarQube Servers

- Add Server
- Add Token
- Name: sonar-server

Manage Jenkins → Tools:
- Add Sonar Scanner

========================
🛡️ TRIVY INSTALLATION
========================

sudo apt install wget apt-transport-https gnupg -y

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo deb https://aquasecurity.github.io/trivy-repo/deb stable main | \
sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt update
sudo apt install trivy -y

Test:
trivy image nginx

========================
🔎 OWASP DEPENDENCY CHECK
========================

Already installed via plugin.

Configure:
Manage Jenkins → Tools → Dependency Check

========================
🐳 DOCKER APP BUILD
========================

Create Dockerfile:

FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]

Build:
docker build -t myapp .

Run:
docker run -d -p 80:3000 myapp

========================
🔁 JENKINS PIPELINE
========================

New Item → Pipeline → Paste below:

pipeline {
    agent any

    tools {
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/your-repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp .'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=myapp \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image myapp'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop myapp || true
                docker rm myapp || true
                docker run -d -p 80:3000 --name myapp myapp
                '''
            }
        }
    }
}

========================
🔗 GITHUB WEBHOOK SETUP
========================

GitHub Repo → Settings → Webhooks → Add:

Payload URL:
http://<EC2-IP>:8080/github-webhook/

Content Type:
application/json

Now every push triggers pipeline automatically.

========================
🌐 NGINX + DOMAIN SETUP
========================

sudo apt install nginx -y

sudo nano /etc/nginx/sites-available/myapp

Paste:

server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
    }
}

Enable:

sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

Test:
sudo nginx -t

Restart:
sudo systemctl restart nginx

========================
📸 IMAGE PLACEHOLDERS
========================

[ADD SCREENSHOT: Jenkins Dashboard]
[ADD SCREENSHOT: SonarQube Dashboard]
[ADD SCREENSHOT: Pipeline Success]
[ADD SCREENSHOT: Trivy Scan Output]

========================
⚠️ COMMON ERRORS
========================

Docker permission:
sudo usermod -aG docker $USER
sudo reboot

Jenkins restart:
sudo systemctl restart jenkins

SonarQube logs:
docker logs sonarqube

========================
💼 LINKEDIN POST (COPY)
========================

🚀 Built a Complete 3-Tier DevSecOps CI/CD Pipeline on AWS!

Tech Stack:
- Jenkins
- Docker
- SonarQube
- Trivy
- OWASP Dependency Check
- AWS EC2

✨ Features:
✔ Automated CI/CD pipeline  
✔ Code quality analysis using SonarQube  
✔ Security scanning with Trivy & OWASP  
✔ Dockerized deployment  
✔ GitHub webhook integration  

This project helped me understand real-world DevSecOps workflows including automation, security, and deployment.

#DevOps #DevSecOps #Jenkins #Docker #AWS #CyberSecurity

############################################################
✅ FINAL RESULT:
FULLY AUTOMATED, SECURE CI/CD PIPELINE 🚀
############################################################