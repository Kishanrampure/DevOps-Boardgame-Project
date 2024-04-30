
# DevOps - Petclinic - Project
Implementing a DevOps CI/CD pipeline for the Sample PetClinic Application in a production level environment.

Understanding the Spring Petclinic application with a few diagrams

![image](https://github.com/Kishanrampure/DevOps-Petclinic-Project/assets/121344253/87ec1469-3a06-4023-addc-139165de83b0)


## Prerequisites
- Install JKD 17
- Install Jenkins
- Install Trivy 
- Install Git
- Install Docker and Docker Compose
- Sonarqube
## #Installation instructions are below.

Install Prerequisites
```bash
  #!/bin/bash

###################################################################
#JDK installation 
###################################################################
sudo apt update
sudo apt install openjdk-17-jdk -y

###################################################################
#Docker installation 
###################################################################
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl enable docker
sudo apt install docker-compose -y


###################################################################
#Jenkins installation 
###################################################################
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt-get install jenkins -y
sudo usermod -aG root jenkins
sudo chmod 777 /var/run/docker.sock
sudo systemctl enable jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

###################################################################
#Git installation 
###################################################################
sudo apt install git-all -y

###################################################################
#Trivy installation 
###################################################################
sudo apt-get install wget apt-transport-https gnupg lsb-release -y

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

###################################################################
#Sonarqube installation 
###################################################################
# Install Postgresql 15
sudo apt upgrade -y

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null

sudo apt update
sudo apt-get -y install postgresql postgresql-contrib
sudo systemctl enable postgresql

sudo passwd postgres
su - postgres
###################################################################
#Create Database for Sonarqube
createuser sonar
psql 

ALTER USER sonar WITH ENCRYPTED password 'sonar';
CREATE DATABASE sonarqube OWNER sonar;
grant all privileges on DATABASE sonarqube to sonar;
\q
exit

###################################################################
#Increase Limits
#Paste the below values at the bottom of the file
sudo vim /etc/security/limits.conf
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
#Exit And Save

###################################################################
#Paste the below values at the bottom of the file
sudo vim /etc/sysctl.conf
vm.max_map_count = 262144
#Exit And Save

###################################################################
#Reboot to set the new limits
sudo reboot

###################################################################
#Install Sonarqube
sudo wget wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.0.89998.zip
sudo apt install unzip
sudo unzip sonarqube-10.5.0.89998.zip -d /opt
sudo mv /opt/sonarqube-10.5.0.89998/ /opt/sonarqube
sudo groupadd sonar
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R

###################################################################
#Update Sonarqube properties with DB credentials
#Find and replace the below values, you might need to add the sonar.jdbc.url 
sudo vim /opt/sonarqube/conf/sonar.properties

sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
#Exit And Save

###################################################################
#Create service for Sonarqube
#Paste the below into the file

sudo vim /etc/systemd/system/sonar.service

[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
#Exit And Save

###################################################################
#Start Sonarqube and Enable service
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
sudo tail -f /opt/sonarqube/logs/sonar.log

###################################################################
#Access the Sonarqube
http://<IP_ADDRESS>:9000
exit
```

## AWS CLI And GCP installation (Optional) | If needed
 - [Setup GCP CLI](https://cloud.google.com/sdk/docs/install)
 - [Setup AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

## Setup Jenkins And SonarQube 

 - [Setup Jenkins (DigitalOcean Link)](https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-ubuntu-22-04)

 - [Setup SonarQube (DigitalOcean Link)](https://www.digitalocean.com/community/tutorials/how-to-ensure-code-quality-with-sonarqube-on-ubuntu-18-04)


## Install Plugins on Jenkins

- Eclipse Temurin installerVersion
- Maven Integration
- Pipeline Maven Integration
- OWASP Dependency-Check
- SonarQube Scanner
- Build Timestamp
- Git Push
- Docker
- Docker Commons
- Docker Pipeline

## If Necessary (Optional)
- Kubernetes Client API
- Kubernetes Credentials
- Kubernetes
- Kubernetes CLI
- Google Kubernetes Engine
- Google Cloud Platform SDK :: Auth
- AWS Credentials
- Amazon Web Services SDK :: All

#
# Setup credentials on jenkins

### Generate tokens in SonarQube and GitHub, and then copy them.


![Sonarqube token copy](https://github.com/Kishanrampure/DevOps-Petclinic-Project/assets/121344253/57030d3a-d3fe-46a0-850b-93cc660734ee)

##

![Github Token Copy](https://github.com/Kishanrampure/DevOps-Petclinic-Project/assets/121344253/0ac85c7f-6388-4c57-bd27-86f2285ec953)
###
#### Click on + Add credentials
###

![Jenkins Create Cred](https://github.com/Kishanrampure/DevOps-Petclinic-Project/assets/121344253/a462880f-38fd-4163-abc7-607ee38aecf7)

## 

![Jenkins cred page](https://github.com/Kishanrampure/DevOps-Petclinic-Project/assets/121344253/c9c67fc8-23ff-4387-9e56-ee82483d66ae)

## Setup Jenkins Tools

| Tool | Name | Versions ~  |
| ----------------- | | ----------------- |------------------------------------------------------------------ |
| JDK | jdk |Install from adoptium.net - 17 |
| SonarQube Scanner installations | sonar-scanner |Install automatically - 5.0.1 |
| Maven installations | M3 |Install from Apache - 3.9.6 |
| Dependency-Check installations | OWAPS |Install from github.com - 9.1.0 |









