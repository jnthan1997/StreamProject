
 
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
 - Login to root user to create new user and add to sudoer
   ```bash
   su root
   
   #add newuser and fillup credentials including new password
   adduser $username #enter username

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

 - 


