# ☕ Java Web App – CI/CD Deployment using Jenkins, Maven & Tomcat (AWS EC2)

This project demonstrates a **fully automated CI/CD pipeline** for deploying a **Java web application** using **Jenkins**, **Maven**, and **Tomcat** hosted on **AWS EC2**.  
It follows a **two-server architecture** — one for Jenkins (automation) and one for Deployment (Tomcat hosting).

---

## 🧱 Architecture Overview

| Server Role | Description | Tools/Software | Purpose |
|--------------|-------------|----------------|----------|
| 🧩 **Jenkins Server** | CI/CD automation | Jenkins, Maven, Java 17 | Pulls code from GitHub, builds using Maven, and deploys to remote Tomcat server |
| 🚀 **Deployment Server** | Web app hosting | Apache Tomcat 9, Java 17 | Hosts the built `.war` file from Jenkins for public access |

---

## 🌐 Workflow

**GitHub → Jenkins (Build) → Deployment Server (Tomcat)**  
1. Developer pushes code to **GitHub**.  
2. **Jenkins** automatically clones the repo and builds the `.war` file using **Maven**.  
3. The generated `.war` file is then deployed to the **Tomcat server** on the **Deployment EC2 instance**.  
4. The Java app is accessible via the **Tomcat web interface**.

---

## ⚙️ Step-by-Step Setup Guide

### 🖥️ 1. Launch Two EC2 Instances
- Create two Ubuntu EC2 instances:
  - **Jenkins Server**
  - **Deployment Server**
- Open required ports:
  - `22` → SSH  
  - `8080` → Tomcat  
  - `8081` (optional) → Jenkins UI  

---

### ⚙️ 2. Configure Jenkins Server

```bash
# Connect to Jenkins EC2
ssh -i "key.pem" ubuntu@<jenkins-server-ip>

# Update packages
sudo apt update -y

# Install Java & Maven
sudo apt install openjdk-17-jdk -y
sudo apt install maven -y

# Install Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update -y
sudo apt install jenkins -y

# Start Jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
```
Access Jenkins UI:
```bash
http://<jenkins-server-ip>:8081
```
## 🧾 3. Setup GitHub Repository

Create a new repository (e.g., **java-maven-tomcat-deploy**).

Push your Java project with `pom.xml` and `src/` folder.

### Example Commands:
```bash 
git init
git remote add origin https://github.com/<username>/java-maven-tomcat-deploy.git
git add .
git commit -m "Initial commit"
git push origin main
```

## 🔐 4. Add SSH Credentials in Jenkins

Navigate to **Manage Jenkins → Credentials → Global → Add Credentials**

Choose **SSH Username with private key**

### Add the following details:

- **ID:** deploy-key  
- **Username:** ubuntu  
- **Private key:** Paste your `.pem` key contents


## 🚀 5. Setup Deployment Server (Tomcat)
```bash
# Connect to deployment EC2
ssh -i "key.pem" ubuntu@<deployment-server-ip>

# Update & install Java
sudo apt update -y
sudo apt install openjdk-17-jdk -y

# Install Tomcat
sudo apt install tomcat9 tomcat9-admin -y

# Start Tomcat
sudo systemctl enable tomcat9
sudo systemctl start tomcat9
```
### Access Tomcat:
```bash
Open your browser and visit:  
**http://<your-server-ip>:8080**
```

## 🧩 6. Jenkins Pipeline Configuration

**jenkinsfile**
```bash
pipeline {
    agent any

    environment {
        SERVER_IP    = '13.200.222.229'
        SSH_CRED_ID  = 'node-key'
        TOMCAT_PATH  = '/var/lib/tomcat10/webapps'
        TOMCAT_SVC   = 'tomcat10'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/iam-ganeshjadhav/Student-Registration-java-form-using-jenkins-maven.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent([SSH_CRED_ID]) {
                    sh """
                        WAR_FILE=\$(ls target/*.war | head -n 1)
                        scp -o StrictHostKeyChecking=no \$WAR_FILE ubuntu@${SERVER_IP}:/tmp/
                        ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} '
                            sudo rm -rf ${TOMCAT_PATH}/*
                            sudo mv /tmp/*.war ${TOMCAT_PATH}/ROOT.war
                            sudo chown tomcat:tomcat ${TOMCAT_PATH}/ROOT.war
                            sudo systemctl restart ${TOMCAT_SVC}
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "App deployed! Visit: http:///"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
```
## ⚙️ 7. Create and Configure a Jenkins Job

Go to **Jenkins Dashboard → New Item → Pipeline**

### Configuration Steps:

- **Enter project name:** `Java-Maven-Tomcat-Deploy`  
- Under **Pipeline Definition**, select **Pipeline script from SCM**  
- **SCM:** Git  
- **Repository URL:** `https://github.com/<username>/java-maven-tomcat-deploy.git`  
- **Branch:** `main`  
- **Script Path:** `Jenkinsfile`  

Click **Save** ✅  
Then hit **Build Now** 🎯  

Jenkins will automatically:  
- Clone your GitHub repository  
- Build your WAR file using Maven  
- Deploy it to your Tomcat server 🚀  


## 🔔 8. Enable GitHub Webhook (Automation Trigger)

To automatically trigger Jenkins when you push to GitHub:

Go to **GitHub Repo → Settings → Webhooks → Add Webhook**

**Payload URL:**
```bash
http://<jenkins-server-ip>:8080/github-webhook/
```
**Content type:** application/json

Select **"Just the push event"**

Click **Add Webhook**

Now every GitHub push will automatically trigger your Jenkins build 🔁

## ✅ 9. Verify Deployment Output

After Jenkins build is successful, visit:
```bash
http://<deployment-server-ip>:8080/<your-app-name>/
```
You should see your Java web application running successfully 🎉

## 📸 Screenshot Gallery

## 📸 Screenshot Gallery

| Step | Description | Screenshot |
|------|--------------|-------------|
| 1️⃣ | Jenkins Build Success | ![Jenkins Build Success](images/jenkins-build-success.png) |
| 2️⃣ | Maven Build Stage | ![Maven Build Stage](images/maven-build.png) |
| 3️⃣ | Tomcat Deployment Complete | ![Tomcat Deployment Complete](images/tomcat-deploy.png) |
| 4️⃣ | Final Java App Running | ![Final Java App Running](images/final-app.png) |


## 🧠 Technologies Used

| Technology | Purpose |
|-------------|----------|
| ☕ **Java 17** | Application development |
| 🏗️ **Maven** | Build automation |
| 🧰 **Jenkins** | Continuous Integration & Deployment |
| 🐱 **GitHub** | Version Control |
| 🐱‍🏍 **Tomcat 9** | Java Application Server |
| ☁️ **AWS EC2** | Cloud Hosting |


## 🧑‍💻 Author

👨‍💻 **Ganesh Jadhav**  
💼 AWS | DevOps | Java Developer  
🔗 [LinkedIn](https://www.linkedin.com/in/your-linkedin-profile)  
📧 **jadhavg9370@gmail.com**

⭐ If you found this project helpful — don’t forget to give it a ⭐ on GitHub!

## 🙏 Acknowledgment

Special thanks to **Trupti Ma’am** for her constant guidance and support throughout this project ❤️


## 🏁 Final Output Preview
**🎉 Java Web Application Deployed Successfully!**

✅ **Build:** SUCCESS  
🚀 **Deployment:** Completed on Tomcat  
🌍 **Access URL:** `http://<deployment-server-ip>:8080/<your-app-name>`  
📦 **Trigger Type:** GitHub Webhook (Auto)

## 📦 Repository Structure
```bash
java-maven-tomcat-deploy/
├── WebContent/ # Web application static files
│ └── (HTML, CSS, JS files)
├── src/ # Java source code
│ └── main/java/...
├── yaml/ # YAML configuration files
├── README.md # Project documentation
├── jenkinsfile # Jenkins pipeline configuration
├── mysql-connector.jar # MySQL JDBC driver
├── pom.xml # Maven configuration file
├── student.sql # Database schema / sample data
├── tomcat-rds-db01 - Copy.txt # Tomcat + RDS connection details
└── xyz # Miscellaneous / placeholder
```
## 🎯 Project Status

✅ **CI/CD:** Fully Functional  
🚀 **Auto Deployment:** Via GitHub Webhook  
☁️ **Hosting:** AWS EC2  
🧩 **Build Tools:** Maven + Jenkins + Tomcat
