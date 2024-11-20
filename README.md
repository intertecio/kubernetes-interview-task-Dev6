# Kubernetes Blue-Green Deployment with Jenkins

This is a readme that walks you through the steps of setting up a Jenkins and deploying it in a Kubernetes cluster using Minikube as well as setting a pipeline that  performs a blue-green deployment of a Spring Boot application.

## Prerequisites

1. **Minikube**: A tool for running Kubernetes clusters locally.
    - [Install Minikube](https://minikube.sigs.k8s.io/docs/)

2. **kubectl**: Command-line tool for interacting with Kubernetes clusters.
    - [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

3. **Docker**: For building Docker images.
    - [Install Docker](https://docs.docker.com/get-docker/)

4. **Helm**: Helm is a package manager for Kubernetes that can simplify the deployment process.
    - [Install Helm](https://helm.sh/docs/intro/install/)

---

## Step 1: Setting up the Minikube Kubernetes Cluster

1. **Start Minikube**:
   Open a terminal window and start Minikube:

   ```bash
   minikube start
   ```

2. **Check Minikube status**:
   Verify that your Kubernetes cluster is running:

   ```bash
   kubectl cluster-info
   ```

   This should show the address of your Kubernetes cluster.

---

## Step 2: The installation of Jenkins on Kubernetes

1. **Install Jenkins via Helm**:
   I chose to use Helm because it simplifies the deployment process. And I used the following [guide](https://www.jenkins.io/doc/book/installing/kubernetes/#install-jenkins-with-helm-v3). I also create kubernetes files for the service account, namespace, persistent volume and the persistent volume claim.

    - Create the Jenkins namespace
       ```bash
       kubectl apply -f k8s/jenkins/jenkins-ns.yaml
       ```
    - Create the Jenkins service account
        ```bash
       kubectl apply -f k8s/jenkins/jenkins-sa.yaml
       ```
    - Create the Jenkins PV and PVC
        ```bash
       kubectl apply -f k8s/jenkins/jenkins-volume.yaml
       ```
    - Add the Jenkins Helm repository:

      ```bash
      helm repo add jenkins https://charts.jenkins.io
      helm repo update
      ```

    - Install Jenkins using Helm:

      ```bash
      chart=jenkinsci/jenkins
      helm install jenkins -n jenkins -f  k8s/jenkins/helm/jenkins-values.yaml $chart
      ```
    
    - I modified the values.yaml file to include the plugins and tools that I need and to use the pvc, namespace and service account that were created.

2. **Access Jenkins**:
   First, retrieve the Jenkins password:

   ```bash
   kubectl get secret --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode
   ```

   To open Jenkins we can use the following command:

   ```bash
   minikube service jenkins --url -n jenkins  
   ```

   We can log in with the username `admin` and the password that was retrieved earlier.

---

## Step 3: The Spring Boot Application

1. **Create a Spring Boot Application**:
   For this example, I created simple Spring Boot application that can be packaged as a Docker container that has one simple endpoint as well as health endpoints.
2. **Dockerize the Application**:
   To dockerize the application we can use the simple maven command 

   ```bash
    mvn spring-boot:build-image
   ```
---

## Step 4: Kubernetes Deployment Manifests

1. To create a blue-green deployment we need to create 2 deployment files. One for the blue and one for the green
2. To keep things clean we create a separate namespace
    ```bash
    kubectl apply -f k8s/app/namespace.yaml
      ```
3. To simulate a blue-green deployment, we can switch traffic by updating the service to point to the new version.
   ```bash
   kubectl apply -f k8s/app/service.yaml
   ```
- First, update the service to point to the green version:

  ```bash
  kubectl patch service spring-boot-app -n app -p ' { "spec" : { "selector" : { "version" : "green" } } }'
  ```

- After testing the green version, you can switch back to the blue version by running:

  ```bash
  kubectl patch service spring-boot-app -n app -p ' { "spec" : { "selector" : { "version" : "blue" } } }'
  ```

---

## Step 5: Automating with Jenkins

1. **Create a Jenkins Pipeline**:
   In Jenkins, create a new pipeline job, add this repository along with a token from github to have access and tell it to use the Jenkinsfile

2. **Run the Pipeline**:
   Trigger the Jenkins pipeline and check the specific steps that need to be executed

## Step 6: Possible improvements

1. Integrate Jenkins with Docker Hub (or any private registry) to automate the image building and pushing process. 
2. Managing the blue and green deployment using static YAML files can be cumbersome, especially for larger applications with multiple services and environments. So it's best to move to helm
3. Move this whole solution to a cloud provider and use IaC such as Terraform
4. Set up a Continuous Deployment pipeline that triggers automatically on every Git commit, merge, or pull request.