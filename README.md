---

COMPLETE DEVSECOPS 3-TIER CI/CD PIPELINE (AWS + JENKINS)

---

## AWS EC2 SETUP

Launch EC2:

- Ubuntu 22.04
- minimum 4GB memory machine
- 12GB storage

Security Group Inbound Rules:

- 22 → SSH
- 8080 → Jenkins
- 9000 → SonarQube
- 80 → Application
- 443 → HTTPS (optional)

---

## SYSTEM SETUP

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

security->Edit inbound port to 8080

http://EC2-IP:8080

Get password:
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

- Install suggested plugins
- Create admin user

---


## SONARQUBE SETUP (DOCKER)

- Architecture Overview

Developer → GitHub → Jenkins (EC2-1) → SonarQube (EC2-2) → Quality Gate

- Docker setup

```bash
sudo apt update && sudo apt upgrade -y
sudo systemctl start docker
sudo systemctl enable docker
```
- Configure Kernel Parameters

```bash
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w fs.file-max=65536
```

- Make these changes permanent
```bash
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
echo "fs.file-max=65536" | sudo tee -a /etc/sysctl.conf
```

- Apply the configuration:

```bash
sudo sysctl -p
```
- Pull SonarQube Docker Image 

```bash
docker pull sonarqube:lts
```

- Run SonarQube Container
```bash
docker run -d \
 --name sonarqube \
 -p 9000:9000 \
 --restart always \
 sonarqube:lts
```

- Check container status:
```bash
docker ps
```

- Open Required Port
```bash
sudo ufw allow 9000
sudo ufw reload
```

---


========================
🔐 CONNECT SONARQUBE WITH JENKINS
========================

Manage Jenkins → System → SonarQube Servers

- Add Server
- Add Token
- Name: sonar-server

Manage Jenkins → Tools:

- Add Sonar Scanner

--- 

## TRIVY INSTALLATION

```bash
sudo apt-get install wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

---


## OWASP DEPENDENCY CHECK

Manage Jenkins → Plugins → Install:

- SonarQube Scanner
- Sonar Quality Gates
- OWASP Dependency-Check
- Docker

Restart Jenkins after install.

# jenkins-SonarQube integration

Add this in SonarQube

Configure:
Adminstration → Configuration → webhook->create

url : http://EC2_PUBLIC_KEY:8080/sonarqube-webhook/

# Generate Tokens



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
