# Jenkins
Introduction to Jenkins. Setting up a Pipeline using Jenkins, Sonarqube and Nexus

## Introduction

In Devops engineering everything is automated.  We automate the CI/CD Pipeline  using the Jenkinsfile

In this project, we are going to learn how to automate pipeline setup with Jenkinsfile.

- Jenkinsfile  defines the stages in CI/CD pipeline

- Jenkinsfile is a text file with pipeline DSL (domain specific language) syntax

-  Similar to groovy but you do not have to understand or write groovy to understand or write a Jenkinsfile.

- Jenkinsfile has two syntax
  - Scripted 
  - Declarative

- Scripted is outdated now Declarative is the way forward now.


## Pipeline Concepts

- Pipeline is the main block of code

  - Jenkins will execute everything within that block
  - node/agent (you can define where the pipeline will be executed on which node or which agent)

- stages define the different stages of the pipeline, such as Build, Test, and Deploy.
Each stage contains specific steps that should be executed for that stage.
  - in stage you have step (eg maven install or git pull or upload artifact or any steps you want to execute from your pipeline)

- The post block defines actions to be taken after the pipeline run, such as notifications or clean-up actions.
### Sample Code  

```
pipeline {
    agent any  // Run on any available agent (Jenkins slave/agent)
    
    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                // Add build commands here (e.g., compile code, package application)
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                // Add test commands here (e.g., run unit tests, integration tests)
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying the application...'
                // Add deployment commands here (e.g., deploy to a server)
            }
        }
    }
    
    post {
        always {
            // Clean up, notifications, or other actions to be performed regardless of the build result
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

```

In this example, we're echoing some messages to represent build, test, and deployment actions.  We are going to replace these with actual build, test, and deployment steps relevant to our project.

- Environment Variables:
Environment variables are specific to a stage or step and can be accessed using the environment block.

```
pipeline {
    agent any

    stages {
        stage('Example') {
            environment {
                ENV_VARIABLE = 'some value'
            }
            steps {
                echo "Environment variable value: ${ENV_VARIABLE}"
            }
        }
    }
}

```
Project 1
In the project we aregoing to write the pipeline to

- Fetch the code
- Build the code 
- Run some tests.

Pay attention to opening and closing of curly braces.

## Steps for CI pipeline Setup 

- Jenkins Server

- Nexus Server

- Sonarqube Server (All three of them will be set up using User data Scripts(bash scripts))

- Security group

- Install neccessary 

- Integrate 
  - nexus
  - Sonarqube with Jenkins

- Write and execute pipeline script

- Set notification

## 

### Jenkins Server

[Jenkins user documentation](https://www.jenkins.io/doc/)


- Launch an Ec2 Instance
  - Ubuntu 20.04 LTS
![os](./images/Jenkins-os.png)

- Instance type
  - t2-small

- Keypair

- Security Group rules
![SG](./images/jenkins-SG.png)

- Launch instance

- Connect to instance 
  - Run the following commands as root


```
sudo apt update

sudo apt install openjdk-11-jdk -y

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null


echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update

sudo apt-get install jenkins -y

```
- Check the status of jenkins

`systemcttl status jenkins`

![status](./images/jenkins-status.png)

- The home directory of jenkins is in `/var/lib/jenkins`

![jenkins](./images/jenkins-directory.png)

- There is a jenkins user called Jenkins
![jenkins](./images/jenkins-user.png)

- Take public IP:8080 of Jenkins server to start to work with 

I had an error at this point.
Security group port for jenkins was set to 80 instead of 8080. 

![jenkins](./images/cat-pwwd.png)

![jenkins](./images/suggested-plugins.png)

![jenkins](./images/create-adminuser.png)

After creating an admin user  Save and finish  and proceed to Jenkins dashboard

![jenkins](./images/jenkins-dashboard.png)

Install node.js plugin


### Install Tools in Jenkins

- Click Manage Jenkins (This is administrative page of Jenkins, only admin user will have access)

- Click on Tools (The tools there are few because they have plugins if you need more tools then you have to install plugins)

- Click add Jdk
  - name - OracleJDK11
  - Java_Home - java path when you connect to Jenkins server
  - ls /usr/lib/jvm/ home directory of jdk on jenkins

![path](./images/jdk-path.png)  

- Click add java again
   - name - OracleJDK8(we do not have that on th server so we will install it)
   - As root run `apt update`
   - `apt upgrade`
   - `apt install openjdk-8-jdk -y`

![path](./images/jdk8-path.png)

- Jenkins UI
![path](./images/add-jdk8nd11.png)

- Maven Tool
  - add the Maven tool
![maven](./images/MAVEN3.png)

- We would have added the git tool but by default, git is already installed in ubuntu. 
- Other distros will have us installing git with the yum package manager. 
   
   















### Nexus server
- Launch Aws Ec2 Instance
- Centos 7 ami
![image](./images/centos-7.png)
- Instance type

  - t2-medium
  
- Key pair

- Create Security group 
  - Nexus-SG

- Advanced settings 
  - User data script

```
#!/bin/bash
yum install java-1.8.0-openjdk.x86_64 wget -y   
mkdir -p /opt/nexus/   
mkdir -p /tmp/nexus/                           
cd /tmp/nexus/
NEXUSURL="https://download.sonatype.com/nexus/3/latest-unix.tar.gz"
wget $NEXUSURL -O nexus.tar.gz
sleep 10
EXTOUT=`tar xzvf nexus.tar.gz`
NEXUSDIR=`echo $EXTOUT | cut -d '/' -f1`
sleep 5
rm -rf /tmp/nexus/nexus.tar.gz
cp -r /tmp/nexus/* /opt/nexus/
sleep 5
useradd nexus
chown -R nexus.nexus /opt/nexus 
cat <<EOT>> /etc/systemd/system/nexus.service
[Unit]                                                                          
Description=nexus service                                                       
After=network.target                                                            
                                                                  
[Service]                                                                       
Type=forking                                                                    
LimitNOFILE=65536                                                               
ExecStart=/opt/nexus/$NEXUSDIR/bin/nexus start                                  
ExecStop=/opt/nexus/$NEXUSDIR/bin/nexus stop                                    
User=nexus                                                                      
Restart=on-abort                                                                
                                                                  
[Install]                                                                       
WantedBy=multi-user.target                                                      

EOT

echo 'run_as_user="nexus"' > /opt/nexus/$NEXUSDIR/bin/nexus.rc
systemctl daemon-reload
systemctl start nexus
systemctl enable nexus

```

- Launch Instance
