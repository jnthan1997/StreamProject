
 
 <h1> Deploy a simple web application using a CI/CD pipeline on your personal device</h1>
  <h3>Disclaimer: The web application that will be deployed for this Stream Project is not owned by me.</h3>
  <img src="./public/assets/pipeline.jpg" alt="Logo" width="100%" height="100%">
  <p> 
   <h1>Project Highlight</h1>
    In this project, we will implement DevOps practices where Jenkins will pull source code from a GitHub repository and automate the process. We will use SonarQube for added security and pull the Quality Gate result before building the Docker image for the web application. Once the build is complete, we will scan the image using Aqua Trivy for additional security. If either the build or the Quality Gate result fails, an email notification will be sent to the developer, and the same will apply for Aqua Trivy results. If both the Quality Gate and Aqua Trivy scans pass, the Docker image will be deployed locally, and the developer will be notified of a successful deployment.
  </p>
  
  <h1> Home Page</h1>
  <img src="./public/assets/home-page.png" alt="Logo" width="100%" height="100%">
  <h1>Netflix like Thumbnails</h1>
  <img src="./public/assets/Sampleimage1.PNG" alt="Logo" width="100%" height="100%">
</div>

# StreamProject
This project is designed to initialize DevOps practices on a local computer or personal device, maximizing resources without relying on Amazon Web Services to avoid additional billing.
The steps and processes for creating this CI/CD pipeline have been tested on AWS and follow the same procedure for Windows Subsystem for Linux (WSL) or VirtualBox.
<div> 
 
## Set up environment using WSL & Oracle Virtualbox
**1. Install WSL on your Windows 10**

  **1.1 Run CMD/Powershell on Administrator mode and run command:**
  ```powershell
  wsl --install
  ```
**1.2 Restart PC and Set Up Linux Distro (e.g: Ubuntu)**
 - Follow prompt and setup username and password.

**1.3 Update Linux Distribution**
```bash
sudo apt update
sudo apt upgrade
```
**2. Install Oracle VirtualBox on your Windows Machine**
 **2.1 Download Oracle VirtualBox installer on the link below:**
 - https://www.oracle.com/sg/virtualization/technologies/vm/downloads/virtualbox-downloads.html

 **2.2 Run Installer**
 - Locate the downloaded installer file (usually in your Downloads folder) and double-click it to run.
   
 **2.3 Follow the Installation Wizard**
 - Follow the prompts in the installation wizard. You can usually accept the default settings unless you have specific preferences.

**2.4 Complete installation**
 - Once the installation is complete, you can launch VirtualBox from your applications menu or desktop shortcut.

**3. Set-up your Linux Ubuntu using OVB**
 - Download Official Image in Ubuntu
 - Create a New Virtual Machine
 - Allocate Memory
 - Create Virtual Hardisk
 - Start Virtual Machine
 - Ensure network is set to NAT Bridge adapter to connect to physical machine

**4. Configure Linux Ubuntu for local deployment**
 - Login to root user to create new user and add user to sud group
   ```bash
   su root
   
   #add newuser and fillup credentials including new password
   adduser $username #enter new username

   #add user to sudoer
   usermod -aG sudo $username

   #switch back to user using
   su $newusername
   ```

 - Configure SSH and Firewall
   ```bash
   #install SSH this will allow us to access virtualbox remotely within same network
   sudo apt-install openssh-server

   #open firewall ssh
   sudo ufw allow openssh
   #to access linux using ssh do the following command line
   ssh $username@<ip-address>
   ```
   
 - Setup Ports for the Tools
   ```bash
   sudo ufw allow 8080 #for Jenkins
   #port 3000 for grafana
   #port 8081 for docker image
   #port 9000 for sonarqube
   #port 9090 for prometheus
   #port 9100 for node exporter
   ```

 - NOTE: All process mentioned in this chapter is also applied in WSL.
 - install git and clone repository
   ```bash
   #Install git
   sudo apt-install git

   #check if git was installed
   git --version

   #clone repository
   git clone https://github.com/jnthan1997/StreamProject

   #check if repo is properly clone
   ls
   #StreamProject folder is supposed show in the directory

   **Now that we are now done setting up our environment we will now set up tools for our practices**

## Docker Installation and SetUp

**1. Uninstall conflicting packages**

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

**2. Install Docker**

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

**3. Install Docker packages**
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**4. Run and Verify Docker**
```bash
docker run hello-world

#will run a simple container image hello-world
#Note: You cannot run docker using your username that has been setup as it is not part of the docker group unless using root account
#add user to docker group inorder to run docker

sudo usermod -aG docker $username

#this will help you to run docker without using root user

sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

#this will add jenkins to docker group for the automation script access docker NOTE: Use this command once the installation of jenkins is donce
#If this command entered it will not be recognize as jenkins is not been made
```

## Build Docker image
Now we are now done our initial Set up we will now explain and build the Docker container based on the dockerfile attached on the repository

 **1. Docker File**
 - Navigate to pwd to StreamProject Directory.
   ```bash
   cd StreamProject

   #Show Directory using:
   $ls
   ```

 - You'll see there is a Dockerfile. Dockerfile is a file that will help us building the container image for our web application.
   ```bash
   vi Dockerfile #open Dockerfile using vi editor
   #or you can use
   sudo nano Dockerfile #open Docker file using nano editor
   ```

   ***Dockerfile***
```bash
#Build application on builder stage
FROM node:16.17.0-alpine as builder
#Set working directory
WORKDIR /app
#copy json package from current directory
COPY ./package*.json .
#Build The application
RUN npm install
#Copy directory to next directory
COPY . .
#Declares Build argument for api key
ARG TMDB_V3_API_KEY
#Pass API KEY variable
ENV VITE_APP_TMDB_V3_API_KEY=${TMDB_V3_API_KEY}
#end point of API
ENV VITE_APP_API_ENDPOINT_URL="https://api.themoviedb.org/3"
#Install only production dependencies
RUN npm run build

#Build application for a runtime stage using nginx
FROM nginx:stable-alpine
#Serve file
WORKDIR /usr/share/nginx/html
#Run Clean state
RUN rm -rf ./*
#Copy Built files from builder stage to NGINX HTML Directory
COPY --from=builder /app/dist .
#Expose nginx to port 80
EXPOSE 80
#Entry point
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```
- NOTE: The dockerfile is structured in two stage the Builder stage and Runtime Stage. Building the application using the nodejs16 image based on alpine linux Setting working directory to /app. Environment variables are set including API key and an API endpoint URL. Deploying or running the build application on stable version of nginx based on alpine linux run time stage, existing files on container image are removed ensuring a clean state and listens on port 80.

 **2. Build Dockerfile using docker build**
  - Before we Build let's make sure to create a account on TMDB as we will using the TMDB API keys to run our application. 
  - After creating the TMBD account and retrieving own API key we will now set a Variable for API key within our WSL or Linux Distro
    ```bash
     #This will export your key within the machine. This is useful for sensitive information like API keys and passwords.
     export TMDB_V3_API_KEY=your_api_key_here
    ```

  - docker build
```bash
   docker build --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY -p 8081:80 -t netflix .
   #docker build - it is a command to build the docker container using dockerfile
   #--build-arg is a command to build arguments within the dockerfile
   #-p 8081:80 - exposing application in port 8081 and listening in port 80 on docker
   #TMBD_V3_API_KEY=$TMDB_V3_APIKEY to pull api key variable within the machine
   #-t $name_of_the_Image - naming the container we are building
   #. - current directory where dockerfile is located
```

- Check the web application by opening it in any web browser of your choice at the address [ip-address]:8081

 ## SonarQube & Trivy Application Installation and Configuration
  
   **1. SonarQube**
   
   - run on shell
     
     ```bash
     docker run -d --name sonarqube -p 9000:9000 sonarqube
     #this will run a latest container image of sonarqube in port 9000
     ```
     
   - Now Open SonarQube in browser [Ip Address:9000]
   - login using Username: admin Password: admin
   - After setting up account **Go to "Administration" > "Security" > "Users"**
   - You'll see administrator on the tokens section click the three dots to generate token
   - Enter the token name and click generate
   - Copy the generate token it is needed for the credentials in Jenkins
   - Now we will now set up the project on the SonarQube
   - **Click Projects > Create Project > Local Project > Add Project Name > Click Next > tick "Use the Global Setting" > Create Project**
   - Create a webhook for Jenkins on the Project created. Go To > Project Settings > Webhook > Create > Add you jenkins URL

   **2. Trivy Installation**
   
 - Install Required Packages
     
        ```bash
        sudo apt-get update
        sudo apt-get install wget apt-transport-https gnupg lsb-release
        ```
 - Add Trivy Repository Key:
   
        ```bash
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
        ```
   
 - Update Package Lists and Install Trivy:
   
        ```bash
        sudo apt-get update
        sudo apt-get install trivy
        ```
   
 - Verify Installation:
   
        ```bash
        trivy --version
        ```
    

  **Now That we Finished setting up the Docker and SonarQube and trivy we will now proceed on setting up Jenkins for CI/CD automation and deployment**

  ## Jenkins Installation and Setup

**1. Install Jenkins**
- Before Installing Jenkins ensure that Java JDK is currently installed on your device by running command:
  ```bash
  java --version
  #it will show your JDK version if not install jdk by:
  sudo apt-get install openjdk-17-jdk
  #This will install and run the JDK, which is required for Jenkins, as Jenkins is developed using Java.
  ```

 - After JDK installation we will now Install the Jenkins:
   ```bash
   # to install LTS or LongTerm Support of Jenkins:
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
   https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
   https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
   /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins
   ```
 
 - After Jenkins installation enable and start jenkins:
   ```bash
   #enable jenkins
   sudo systemctl enable jenkins
   #start jenkins
   sudo systemctl start jenkins
   #check jenkins statu
   sudo systemctl status jenkins
   ```

   **NOTE: Jenkins is running on port 8080 that is why on the earlier step we did we enable firewall port 8080 so we can run Jenkins in browser.
   To open Jenkins open browser of your choice at the address [ip-address:8080]**

   **2. Setup Jenkins**
   
   Now Jenkins is running on the system we will now setup Jenkins Interface
   - Run Jenkins on your browser of your choice [ip-address:8080]
   - Once Open Jenkins will show you the path for it's password
   - Enter password then choose **Install Suggested Plugins**
   - After the installation it will prompt you to page to create a account and specified a URL.
   - Once Done we need to Install plugins needed for our project:
     
     **Go To " Manage Jenkins" > "Plugins" > "Available Plugins" then install the following plugins:**
     
      **Eclipse Temurin Installer**
     
      **SonarQube Scanner**
     
      **NodeJs Plugin**
     
      **Email Extension Plugin**
     
      **Eclipse Temurin Installer**
     
      **SonarQube Scanner**
     
      **NodeJs Plugin**
     
      **Email Extension Plugin**

      **Docker**
     
      **Docker Commons**

      **Docker Pipeline**

      **Docker API**
  
      **Prometheus*
 
     - We will now configure the Plugins that we've been installed
       **Configure Plugins**
      **Go To "Manage Jenkins" > "Tools"**
       - Locate JDK Installation add name "jdk17"
       - Tick Install automaticaly
       - Click "Add Installer" choose "Install From Adoptium"
       - choose a Java version
         
       **Note: The Configuration above is the same process for NodeJS Plugin, SonarQube Scanner, Dependency Check Installation mention plugin can be found on the same tab**
       
      **Configure Jenkins Credentials**
       **Go To "Manage Jenkins" > "Credentials"**
       - 1.Click System
       - 2.Click Global Credentilas
       - 3.Click Add Credentials
         Now before we continue Setup the SonarQube container based on the Guide **SonarQube Application Configuration**
       - 4.On **Kind** section choose **Secret Text** then on **Secret** paste the generated token that we get on SonarQube
       - 5.add ID for the SonarQube toke
       - 6.click Create
     
       - **Now Repeat the process 1 to 6 for the TMDB API Key(Kind: Secret Text), Docker(Kind: Username with Password) and Gmail(Kind: Username with Password)**
    
     **Configure Jenkins other Plugins**
     **Go To "Manage Jenkins" > "System"**
     **Locate "SonarQube servers"**
     - Name: 'sonar-server'
     - URL: [IP Address:9000]
     - Choose the token for sonarqube
       
     **Locate "Extended Email Notification"**
     - SMTP Server: smtp.gmail.com
     - SMTP Port: 465
     - Click Advance > Choose your Gmail Credentials and tick "Use SSL"
     - Scroll Down a bit then click **"Build Triggers"** Tick **"Always"**
    
      **Locate "Email Notification"**
     - SMTP Server: smtp.gmail.com
     - SMTP Port: 465
     - Click Advance
     - Input your Email Address
     - For the password use Application password of google
    
       **CLICK APPLY Then SAVE**
       
     **3. Creating New Pipeline**
       **Go to Dashboard > New Item > Pipeline then OK**
       - Put a Description of the project depend on the Description you want.
       - Next Go to Pipeline and add the Jenkinsfile copy and paste it on the field.(using jenkinsfile will be discussed in the next project)
         
       **(NOTE: Groovy Script within the Jenkinsfile is commented for those less experienced with Groovy Script.)**

     ## Install Monitoring Tool
       
     **1. Install Prometheus**

     Create user dedicated User on Linux For Prometheus and Download the file using wget
     ```bash
      sudo useradd --system --no-create-home --shell /bin/false prometheus
      wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
     ```

     Extract file:

     ```bash
     tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
     cd prometheus-2.47.1.linux-amd64/
     sudo mkdir -p /data /etc/prometheus
     sudo mv prometheus promtool /usr/local/bin/
     sudo mv consoles/ console_libraries/ /etc/prometheus/
     sudo mv prometheus.yml /etc/prometheus/prometheus.yml
     ```
     
     Set Ownership and Create a Systemd unit:
     
     ```bash
     #Ownership
     sudo chown -R prometheus:prometheus /etc/prometheus/ /data/

     #Create SystemD unit
     sudo nano /etc/systemd/system/prometheus.service
     ```

     Add the following content:

     ```bash
     [Unit]
     Description=Prometheus
     Wants=network-online.target
     After=network-online.target

     StartLimitIntervalSec=500
     StartLimitBurst=5

     [Service]
     User=prometheus
     Group=prometheus
     Type=simple
     Restart=on-failure
     RestartSec=5s
     ExecStart=/usr/local/bin/prometheus \
       --config.file=/etc/prometheus/prometheus.yml \
       --storage.tsdb.path=/data \
       --web.console.templates=/etc/prometheus/consoles \
       --web.console.libraries=/etc/prometheus/console_libraries \
       --web.listen-address=0.0.0.0:9090 \
       --web.enable-lifecycle
     
     [Install]
     WantedBy=multi-user.target
     ```

     Enable Prometheus:

     ```bash

     sudo systemctl enable prometheus
     sudo systemctl start prometheus
     ```

     Check Prometheus status:

     ```bash
     sudo systemctl status prometheus
     ```

     After checking the status you may now access Prometheus on your browser [ip-address:9090]
  
     **Prometheus configuration:**
  
     configure Prometheus to scrape metrics and monitor CI/CD pipeline. We need to modify the prometheus.yaml
  
     ```bash
          global:
       scrape_interval: 15s
     
     scrape_configs:
     
       - job_name: 'jenkins'
         metrics_path: '/prometheus'
         static_configs:
           - targets: ['<your-jenkins-ip>:<your-jenkins-port>']
     ```
     
     Reload prometheus configuration without restarting
  
     ```bash
     curl -X POST http://localhost:9090/-/reload
     ```

     **2.Node Exporter Installation**

     Create a system user for Node Exporter and download Node Exporter:

     ```bash
     sudo useradd --system --no-create-home --shell /bin/false node_exporter
     wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
     ```

     Create a systemd unit configuration file for Node Exporter:

     ```bash
     sudo nano /etc/systemd/system/node_exporter.service
     ```

     Add the following content to the node_exporter.service file:

     ```bash
     [Unit]
     Description=Node Exporter
     Wants=network-online.target
     After=network-online.target
     
     StartLimitIntervalSec=500
     StartLimitBurst=5
     
     [Service]
     User=node_exporter
     Group=node_exporter
     Type=simple
     Restart=on-failure
     RestartSec=5s
     ExecStart=/usr/local/bin/node_exporter --collector.logind
     
     [Install]
     WantedBy=multi-user.target
     ```

     Enable, Start and check status of Node Exporter

     ```bash
     sudo systemctl enable node_exporter
     sudo systemctl start node_exporter
     sudo systemctl status node_exporter
     ```

     **3. Install Grafana for metric dashboard**

     Install Dependencies:

     ```bash
     sudo apt-get update
     sudo apt-get install -y apt-transport-https software-properties-common
     ```

     Add GPG key:

     ```bash
     wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
     ```

     Add Grafana repositories:
     ```bash
     wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
     ```

     Update and Install Grafana:

     ```bash
     sudo apt-get update
     sudo apt-get -y install grafana
     ```

     Enable, Start and check Status of Grafana:

     ```bash
     sudo systemctl enable grafana-server
     sudo systemctl start grafana-server
     sudo systemctl status grafana-server
     ```
     Access Grafana Server on browser by using [IP-Address:3000]
     You'll be prompted in Grafana UI and default user and password is: admin

     Import Dashboard in Grafana:

     

     - Click on the "+" (plus) icon in the left sidebar to open the "Create" menu.

     - Select "Dashboard."
   
     - Click on the "Import" dashboard option.
   
     - Enter the dashboard code you want to import (e.g., code 1860).
   
     - Click the "Load" button.
   
     - Select the data source you added (Prometheus) from the dropdown.
   
     - Click on the "Import" button.

    You should now have a Grafana dashboard set up to visualize metrics from Prometheus.

    Grafana is a powerful tool for creating visualizations and dashboards, and you can further customize it to suit your specific monitoring needs.
   
   
   ## Summary of the Project

   What we did in this project is we setup our environment using WSL or Oracle VirtualBox with Linux Ubuntu Image. We Open Ports and install SSH to access the System Remotely. After we set up
   and open ports needed to access tools in browser. We Install Necessary Tools for our project Like Docker, Jenkins, SonarQube, Trivy, Prometheus, Node Exporter and Grafana.
   We Set Up Docker run and build  the sample project Inorder to have a overview of the web application that we will deploy. Next is we export variable within our machine for our API key that we
   retrieved in the TMDB website. This is a practice made to ensure the security of credentials such as password and APi Keys. Next we run SonarQube docker Image and setup its token, webhook for jenkins
   and Project to retrieve quality gate result. after setting up SonarQube we installed trivy for Scanning docker image, next we proceed with the Jenkins Setup install necessarry plugins for our pipeline
   then we install monitoring tools to monitor our CI/CD pipeline using Grafana and Prometheus.

   For troubleshooting steps and Questions, please feel free to message me.
   


