# INTEGRATING HELMS WITH JENKINS

###  Integrating Helm with Jenkins for CI/CD

### Prerequisites

Before you begin, make sure you have:

- An Ubuntu or any Linux system (or VM)

- Jenkins installed and running

- Helm installed

- Kubernetes cluster (e.g. Minikube, EKS, or any other)

- A Git repository (where your code and Helm chart live)

- Docker installed (if building Docker images)

**Step 1:** Jenkins Setup

1 Install Jenkins

If not already installed:

 ```
   sudo apt update
sudo apt install openjdk-11-jdk -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
```
Then start Jenkins:

```
  sudo systemctl enable jenkins
  sudo systemctl start jenkins
```
Access Jenkins at:
  ```
    http://localhost:8080
  ```  
 Unlock Jenkins with:
  ```
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
 ```
 ### Step 2: Determine Helm Binary Path

The Jenkins pipeline needs to know the exact location of Helm.

**On Linux:**
  ```
   which helm
  ```

### Step 3: Create a Jenkins Pipeline
1 Open Jenkins → New Item → PipelineStep 3: Create a Jenkins Pipeline
Give it a name (Helm-CID),choose pipeline **OK**

2 **Set Source to Git Repository**

In the **Pipeline section:**

- Choose Pipeline script from SCM

- Select Git

- Enter your repo URL (e.g., https://github.com/yourname/your-repo.git)

- Enter branch (e.g., main)

###        3 Add a Jenkinsfile in your Git repo

In your repository root, create a file called Jenkinsfile with content like this:

```
  pipeline {
    agent any

    environment {
        HELM_HOME = '/usr/local/bin/helm'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/Nonsocha/Helms-jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker tag myapp:latest myrepo/myapp:latest'
                sh 'docker push myrepo/myapp:latest'
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh "${HELM_HOME} upgrade --install myapp ./chart --values ./chart/values.yaml"
            }
        }
    }
}
```

### Step 4: Run the Pipeline

1 Click **Build Now** in Jenkins.

2 Watch the stages execute:

- Checkout code

- Build Docker image

- Push image

- Deploy to Kubernetes using Helm
### Step 5: Verify Deployment

Run on your terminal:
```
  kubectl get pods
  kubectl get svc
 ```

 ## Step 2 Update helms Chart and Trigger Jenkins Pipeline
### Create a File Name Jenkinsfile at the root directoyry of the repo and add 

```
  pipeline {
    agent any

    environment {
        HELM_HOME = '/usr/local/bin/helm'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/Nonsocha/Helms-jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker tag myapp:latest myrepo/myapp:latest'
                sh 'docker push myrepo/myapp:latest'
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh "${HELM_HOME} upgrade --install myapp ./chart --values ./chart/values.yaml"
            }
        }
    }
}
```
### Step 2: Update Helm Chart and Trigger Jenkins Pipeline

Now that the pipeline is ready, you’ll test it by making a Helm chart change.

1 **Edit** values.yaml

Navigate to your Helm chart folder (e.g., webapp/values.yaml) and open it.
 
 **Change:**
    
  ```
  replicaCount: 1
  ```
  **to**:
    
  ```
  replicaCount: 3
```
 **Save and commit the change:**
 ```
   git add webapp/values.yaml
  git commit -m "Updated replica count to 3"
  git push origin main
 ```
 This push will automatically trigger Jenkins if your pipeline is configured for SCM polling or a webhook.
