
 
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

**3. Instal Docker packages**
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
FROM node:16.17.0-alpine as builder
WORKDIR /app
COPY ./package.json .
COPY ./package-lock.json .
RUN npm install
COPY . .
ARG TMDB_V3_API_KEY
ENV VITE_APP_TMDB_V3_API_KEY=${TMDB_V3_API_KEY}
ENV VITE_APP_API_ENDPOINT_URL="https://api.themoviedb.org/3"
RUN npm run build

FROM nginx:stable-alpine
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY --from=builder /app/dist .
EXPOSE 80
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

   
   
   
   


