# Configuration Management with Ansible — Exercises & Solutions

**Repository to use:** [GitLab – ansible-exercises](https://gitlab.com/twn-devops-bootcamp/latest/15-ansible/ansible-exercises)

---

## Exercise 1: Build & Deploy Java Artifact

**Task:**
- Create an Ansible project that builds a Java application and deploys the built jar artifact to a remote Ubuntu server
- Developers specify their first name as the Linux user to start the application
- Create the Linux user if it doesn't exist on the remote server
- Stop any existing application and remove old jar files before deploying the new one

**Solution:**

**Preparation:**
```sh
# Create an Ubuntu server (Ubuntu version 24.04) on any platform
# Configure ssh access (open port 22, create ssh key access)
# Set "server IP address", "user" and "ssh private key path" in "hosts-deploy-app" file

# Fix the test in the application to allow building it successfully
# In file - src/test/java/AppTest.java, line 22, remove quotes from true to make it boolean

# Install the "acl" package on the ubuntu machine
sudo apt-get update -y
sudo apt-get install -y acl
```

**Implementation:**
```sh
# Implementation in file: 1-build-and-deploy.yaml 

# Execute with hosts-deploy-app host file
ansible-playbook -i hosts-deploy-app 1-build-and-deploy.yaml --extra-vars "linux_user=your-name project_dir=/absolute/path/to/java/gradle/project jar_name=bootcamp-java-project-1.0-SNAPSHOT.jar"
```

---

## Exercise 2: Push Java Artifact to Nexus

**Task:**
- Write a playbook that allows developers to specify a jar file and push it to the team's Nexus repository
- Provide convenience for developers to push successful artifacts after testing

**Solution:**

```sh
# Make sure you have a Nexus repository manager up and running with maven-snapshots repository

# Implementation in file: 2-push-to-nexus.yaml 

# Execute 
ansible-playbook 2-push-to-nexus.yaml --extra-vars "nexus_url=http://nexus-ip:nexus-port nexus_user=admin nexus_password=admin-pass repository_name=maven-snapshots artifact_name=bootcamp-java-project artifact_version=1.0-SNAPSHOT jar_file_path=/absolute/path/to/jar/file/bootcamp-java-project-1.0-SNAPSHOT"
```

**Note:** You can specify any inventory file to avoid warnings, even though we're using localhost.

---

## Exercise 3: Install Jenkins on EC2

**Task:**
- Write Ansible code that creates a new EC2 server and installs Jenkins on it
- Install nodejs, npm and docker for Jenkins builds
- Enable team to spin up a new Jenkins server with 1 Ansible command

**Solution:**

```sh
# Implementation in files: 
# - 3-provision-jenkins-ec2.yaml 
# - 3-install-jenkins-ec2.yaml 

# Execute to provision jenkins server 
ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/path/to/ssh-key/file aws_region=your-aws-region key_name=your-key-pair-name subnet_id=your-subnet-id ami_id=image-id-for-amazon-linux ssh_user=ec2-user" 

# Wait until the server is fully initialised

# Execute to configure jenkins server 
ansible-playbook -i hosts-jenkins-server 3-install-jenkins-ec2.yaml --extra-vars "aws_region=your-aws-region"
```

**Note:** You can parameterize more values or use variables files for flexibility.

---

## Exercise 4: Install Jenkins on Ubuntu

**Task:**
- Re-write your playbook to support both Amazon Linux and Ubuntu OS flavors
- Use include_tasks or conditionals to support multiple platforms
- Provide infrastructure flexibility across different OS platforms

**Solution:**

```sh
# Implementation in files: 
# - 3-provision-jenkins-ec2.yaml 
# - 4-install-jenkins-ubuntu.yaml
# - 4-host-amazon.yaml
# - 4-host-ubuntu.yaml

# To Create and configure Jenkins on Ubuntu EC2 instance
ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/path/to/ssh-key/file aws_region=your-aws-region key_name=your-key-pair-name subnet_id=your-subnet-id ami_id=image-id-for-ubuntu ssh_user=ubuntu"

ansible-playbook -i hosts-jenkins-server 4-install-jenkins-ubuntu.yaml --extra-vars "host_os=ubuntu aws_region=your-aws-region"

# To Create and configure Jenkins on Amazon Linux EC2 instance
ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/path/to/ssh-key/file aws_region=your-aws-region key_name=your-key-pair-name subnet_id=your-subnet-id ami_id=image-id-for-amazon-linux ssh_user=ec2-user"

ansible-playbook -i hosts-jenkins-server 4-install-jenkins-ubuntu.yaml --extra-vars "host_os=amazon aws_region=your-aws-region"
```

**Notes:**
- The only difference in provisioning is the `ami_id` value
- Shared tasks are in `provision-jenkins-ec2.yaml` with OS-specific differences in respective host files
- Dynamic selection based on `host_os` variable

---

## Exercise 5: Install Jenkins as a Docker Container

**Task:**
- Write a playbook that starts Jenkins as a Docker container
- Configure volumes for Jenkins home and Docker socket
- Enable Docker command execution inside Jenkins

**Reference Docker Command:**
```sh
docker run --name jenkins -p 8080:8080 -p 50000:50000 -d \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/local/bin/docker:/usr/bin/docker \
-v jenkins_home:/var/jenkins_home \
jenkins/jenkins:lts
```

**Solution:**

```sh
# Implementation in files: 
# - 3-provision-jenkins-ec2.yaml 
# - 5-install-jenkins-docker.yaml

# Execute to provision the ubuntu Jenkins server
ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/path/to/ssh-key/file aws_region=your-aws-region key_name=your-key-pair-name subnet_id=your-subnet-id ami_id=image-id-for-ubuntu ssh_user=ubuntu"

# Wait until the server is fully initialised

# Execute to configure server to run jenkins as a docker container
ansible-playbook -i hosts-jenkins-server 5-install-jenkins-docker.yaml --extra-vars "aws_region=your-aws-region"
```

---

## Exercise 6: Web server and Database server configuration

**Task:**
- Automate deploying and configuring a web server and database server on AWS
- Create dedicated Ansible server in the same VPC
- Set up Java application server and MySQL database server
- Database server should not be accessible from outside (no public IP)

**Architecture:**
- Ansible control server (public)
- Web server with Java application (public)
- Database server with MySQL (private, VPC-only access)

**Solution:**

**Preparation:**
```sh
# Create AWS key-pairs and set permissions
chmod 400 ~/Downloads/ansible-managed-server-key.pem
chmod 400 ~/Downloads/ansible-control-server-key.pem

# Configure NAT Gateway for private subnet internet access:
# 1. Create NAT gateway "my-nat" in PUBLIC subnet with Elastic IP
# 2. Create route table "my-db-rt" with route: 0.0.0.0/0 -> "my-nat"
# 3. Associate private subnet with "my-db-rt" route table
# 4. Note public and private subnet IDs for playbook execution

# Update ansible.cfg file:
enable_plugins = amazon.aws.aws_ec2
remote_user = ubuntu
private_key_file = /home/ubuntu/ansible-managed-server-key.pem
```

**Implementation:**
```sh
# Implementation in files: 
# - 6-provision-ansible-server.yaml, 6-configure-ansible-server.yaml
# - 6-inventory_aws_ec2.yaml, 6-provision-app-servers.yaml
# - 6-configure-app-servers.yaml, 6-vars.yaml

# Execute playbook to provision ansible control server
ansible-playbook 6-provision-ansible-server.yaml --extra-vars "aws_region=eu-west-3 key_name=ansible-control-server-key subnet_id=public-subnet-id ami_id=ami-id"

# Configure ansible control server with tools and files
ansible-playbook -i 6-inventory_aws_ec2.yaml 6-configure-ansible-server.yaml

# SSH into ansible control server
ssh -i ~/Downloads/ansible-control-server-key.pem ubuntu@ansible-server-public-ip

# Provision web and database servers
ansible-playbook 6-provision-app-servers.yaml --extra-vars "aws_region=eu-west-3 key_name=ansible-managed-server-key subnet_id_web=public-subnet-id subnet_id_db=private-subnet-id ami_id=ami-id"

# Configure both servers
ansible-playbook -i 6-inventory_aws_ec2.yaml 6-configure-app-servers.yaml
```

**Verification:**
Access the application at `http://web-server-public-ip:8080` (ensure port 8080 is open)

---

## Exercise 7: Deploy Java MySQL Application in Kubernetes

**Task:**
- Dockerize the Java MySQL application and deploy to K8s cluster
- Create K8s configuration files for deployments, services, configMap and Secret
- Deploy nginx-ingress controller and create ingress for web access
- Automate the entire deployment process with Ansible

**Solution:**

**Preparation:**
```sh
# Dockerize the application using Dockerfile from solutions branch
# Build and push image as: "your-docker-hub-id/demo-app:java-mysql-app"

# Create K8s cluster (Minikube, LKE, EKS) and get kubeconfig file
export KUBECONFIG=/path/to/kube/config/file

# Set ingress host in kubernetes-manifests/exercise-7/java-app-ingress.yaml
```

**Implementation:**
```sh
# Implementation in files: 
# - 7-deploy-on-k8s.yaml
# - kubernetes-manifests/exercise-7/*.yaml

# Execute playbook to deploy K8s manifests
ansible-playbook 7-deploy-on-k8s.yaml --extra-vars "docker_user=your-dockerhub-user docker_pass=your-dockerhub-password"
```

**Note:** If you encounter nginx-controller-admission webhook errors:
```sh
kubectl get ValidatingWebhookConfiguration
kubectl delete ValidatingWebhookConfiguration {name}
```

---

## Exercise 8: Deploy MySQL Chart in Kubernetes

**Task:**
- Deploy MySQL with 3 replicas using a Helm chart for high availability
- Replace the single MySQL instance from Exercise 7
- Use Ansible to automate the Helm chart deployment

**Solution:**

**Preparation:**
```sh
# Set ingress host in kubernetes-manifests/exercise-8/java-app-ingress.yaml
export KUBECONFIG=/path/to/kube/config/file

# Remove existing MySQL deployment from exercise 7
kubectl delete deployment mysql-deployment
```

**Implementation:**
```sh
# Implementation in files: 
# - 8-deploy-on-k8s.yaml
# - kubernetes-manifests/exercise-8/*.yaml

# Execute playbook to deploy MySQL chart
ansible-playbook 8-deploy-on-k8s.yaml --extra-vars "docker_user=your-dockerhub-user docker_pass=your-dockerhub-password"
```

---

## Solutions in Code Structure

```
solutions-in-code/
├── 1-build-and-deploy.yaml
├── 2-push-to-nexus.yaml  
├── 3-provision-jenkins-ec2.yaml
├── 3-install-jenkins-ec2.yaml
├── 4-install-jenkins-ubuntu.yaml
├── 4-host-amazon.yaml
├── 4-host-ubuntu.yaml
├── 5-install-jenkins-docker.yaml
├── 6-provision-ansible-server.yaml
├── 6-configure-ansible-server.yaml
├── 6-inventory_aws_ec2.yaml
├── 6-provision-app-servers.yaml
├── 6-configure-app-servers.yaml
├── 6-vars.yaml
├── 7-deploy-on-k8s.yaml
├── 8-deploy-on-k8s.yaml
├── kubernetes-manifests/
│   ├── exercise-7/
│   └── exercise-8/
├── hosts
└── ansible.cfg
```

## Key Learning Outcomes

- **Application Deployment:** Automated Java application building and deployment
- **Infrastructure Provisioning:** EC2 server creation and configuration
- **Multi-Platform Support:** Cross-platform compatibility (Amazon Linux, Ubuntu)  
- **Containerization:** Docker-based deployments and container orchestration
- **Database Management:** MySQL setup and high-availability configurations
- **Kubernetes Automation:** Container orchestration with automated deployments
- **Security Best Practices:** Private networking, NAT gateways, and secure access patterns
