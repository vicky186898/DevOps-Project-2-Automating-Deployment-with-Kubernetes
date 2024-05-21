                               DEVOPS PROJECT-2 
You are hired as a DevOps Engineer for Analytics Pvt Ltd. This company is a product-based 
organization which uses Docker for their containerization needs within the company. The 
final product received a lot of traction in the first few weeks of launch. Now with the 
increasing demand, the organization needs to have a platform for automating deployment, 
 scaling and operations of application containers across clusters of hosts. As a DevOps 
Engineer, you need to implement a DevOps lifecycle such that all the requirements are 
implemented without any change in the Docker containers in the testing environment.  
Up until now, this organization used to follow a monolithic architecture with just 2 
developers. The product is present on: https://github.com/hshar/website.git  
Following are the specifications of the lifecycle:  
1. Git workflow should be implemented. Since the company follows a monolithic 
architecture of development, you need to take care of version control. The release should 
happen only on the 25th of every month.  
2. CodeBuild should be triggered once the commits are made in the master branch. 
3. The code should be containerized with the help of the Dockerfile. The Dockerfile 
should be built every time if there is a push to GitHub. Create a custom Docker 
image using a Dockerfile.  
4. As per the requirement in the production server, you need to use the Kubernetes 
cluster and the containerized code from Docker Hub should be deployed with 2 
replicas. Create a NodePort service and configure the same for port 30008.  
5. Create a Jenkins Pipeline script to accomplish the above task. 
6. For configuration management of the infrastructure, you need to deploy the 
configuration on the servers to install the necessary software and configurations.  
7. Using Terraform, accomplish the task of infrastructure creation in the AWS cloud 
provider.  
Architectural Advice:  
Software to be installed on the respective machines using configuration management.  
Worker1: Jenkins, Java  
Worker2: Docker, Kubernetes  
Worker3: Java, Docker, Kubernetes Worker4: Docker, Kubernetes 



Solution :-
Steps: - 
Machine 1: Jenkins_Terraform_Ansible 
Machine2: Kmaster 
Machine3: Kslave1 
1. Create an instance for using terraform, name it like (Machine-TAJ). TAJ refers that it 
will have terraform, Jenkins and ansible. 
2. Connect to your instance. 
Terraform- Machine1
3. Install terraform manually. 
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o 
/usr/share/keyrings/hashicorp-archive-keyring.gpg 
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] 
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee 
/etc/apt/sources.list.d/hashicorp.list 
sudo apt update && sudo apt install terraform -y 



4. Create a terraform code to create the infrastructure of all the instances that we 
require on AWS. 
provider "aws" {
  secret_key = "Ur0GCdYFsKMTUIFUgS"
  access_key = "AKIA4MTWLB"
  region = "us-east-2"                     —-# make sure region is correct
}

resource "aws_instance" "K8-M" {
  ami = " ami-07b36ea9852e986ad"
  instance_type = "t2.medium"
  key_name = "new-key"                  —#pass your key
  tags = {
    Name = "Kmaster"                                    #instance 2
  }
}

resource "aws_instance" "K8-S1" {
  ami = " ami-07b36ea9852e986ad"
  instance_type = "t2.micro"
  key_name = "new-key"
  tags = {
    Name = "Kslave1"                                          #Instance3
  }
}
5. Now use terraform init, terraform plan and terraform apply. 
Ansible Machine1
6. Install ansible on this ec2 instance. 
sudo apt update 
sudo apt install software-properties-common 
sudo add-apt-repository --yes --update ppa:ansible/ansible 
sudo apt install ansible -y 

ANSIBLE :-

7. Generate private key on ansible machine by “ssh-keygen” 

8. Now we will go to authorized keys in slave machine and copy the content of id_rsa.pub 
to authorized key. This process is known as password less ssh. 
9. Go to ansible server and go into /etc/ansible/. 

10. Add identities of other slave EC2.  
a. For this nano hosts 
b.  Give the name slave1_k8master and add IP address of slave2_k8slave and 
similarly for production give the IP address of production. 
 
Playbook for Installations of Scripts
11. Create a playbook for the installation part. - name: Installations on Master 
hosts: localhost 
become: true 
tasks: - name: Executing script on master 
script: master.sh
 - name: Installations on Kmaster 
hosts: Kmaster 
become: true 
tasks: 
- name: Executing script on Kmaster 
script: kmaster.sh 
 - name: Installations on Kslave 
hosts: kslave 
become: true 
tasks: 
- name: Executing script on kslave
script: kslave.sh 


12. Now create machine1 installation script. Name it as machine1.sh. 
Java, docker, Jenkins 
Jenkins_terraform_ansible.sh:
sudo apt update
sudo apt-get install openjdk-11-jdk -y
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
sudo apt-get install docker.io -y


For kmaster.sh and kslave.sh we will need same script because slave will also need kubeadm and kubectl when we will paste the token from kmaster.(don’t add sudo kubeadm init command we have to do it manually in master)
sudo apt update
sudo apt install openjdk-11-jdk -y
sudo apt install docker.io -y
 sudo apt upgrade -y
sudo apt install docker.io -y 
sudo apt install -y curl apt-transport-https ca-certificates software-properties-common
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo swapoff -a
sudo apt update
sudo apt install -y kubelet kubeadm kubectl


13. Create slave1 installation script. Name it as k8smaster-jenkins-s1.sh
Java, docker, Kubernetes 
14. Create slave2 installation script. Name it as slave2-k8slave.sh
Java, Docker, Kubelet and kubectl, kubeadm 
15. Run the playbook. 
 

16. Now generate the token in kmaster using kubeadm init command. 
17. Copy the token in the k8s slave machine. 
 


18. Open the Jenkins dashboard, click on manage Jenkins, go to nodes, click on new 
node. 
 

19. Give the name of the node, select the type as permanent agent.
 Description for hosting application. Remote root directory “/home/ubuntu/Jenkins”. 
20.  Launch method via ssh.
In host add private host name.
Add credentials in kind
change with private key and username.
Give an id give username as ubuntu and in
key add the content of .pem file.
Change the host key verfication id and non verifying.



21. Add kslave also in the node with the similar steps given above. 

22. Now create the docker file 
FROM ubuntu 
RUN apt update 
RUN apt-get install apache2 -y 
ADD . /var/www/html 
ENTRYPOINT apachectl -D FOREGROUND 

23. Now create deployment.yaml 
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: custom-deployment 
  labels: 
    app: custom 
spec: 
  replicas: 2 
  selector: 
    matchLabels: 
      app: custom 
  template: 
    metadata: 
      labels: 
        app: custom 
    spec: 
      containers: 
      - name: custom 
        image: dhananjay9/image 
        ports: 
        - containerPort: 80 
 
24. Create a service yaml also. 
apiVersion: v1 
kind: Service 
metadata: 
  name: my-custom-deployment 
spec: 
  type: NodePort 
  ports: 
    - targetPort: 80 
      port: 80 
      nodePort: 30008 
  selector: 
    app: custom 
 
25. Now create a job. 
a). Click on new item.
 Name the job, select the pipeline method.  
b). Add the description “this job will be triggered when master branch 
containing all the fill will be pushed to the GitHub account mentioned below. 
This will clone the new data bron the GitHub account to the 
machine(kmaster) because we will make this job run on kmaster machine. 
So, all the deployment and files will be carried out on kmaster. As you know, 
whatever runs on Kubernetes master, it automatically runs on Kubernetes 
slave, so Kubernetes slave has no other operation in this project.” 
c). Select git and add git repository URL.  
d. In Build triggers select GitHub webhooks. (do the required changes in GitHub 
account also) 
e. Now let's create a pipeline script. 
 
pipeline{ 
    agent none 
    environment { 
        DOCKERHUB_CREDENTIALS=credentials(‘id-of-parameter')  ###Go to Jenkins dashboard   
                                                                                                   credentials  Select Global                                                                                                                                                                                                                                                       
                                                                                                     username and password   of Dockerhub
                                                                                                      it provides us a ID for the same    
                                                                                          5e9bbc62-6d6d-4399-b90d-7cf7ffbb3158                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               
  [ Adding DockerHub credentials in a pipeline is necessary when your pipeline requires access to private DockerHub repositories or when it needs to push Docker images to DockerHub]
 stages{                                                  
        stage('git'){ 
            agent{  
                label 'KMaster' 
            } 
            steps{ 
                git'https://github.com/Intellipaat-Training/Test.git' 
            } 
        } 
        stage('docker') { 
            agent {  
                label 'KMaster' 
            } 
            steps { 
                sh 'sudo docker build . -t dhananjay/image' 
                sh 'sudo echo $DOCKERHUB_CREDENTIALS_PSW | sudo docker login -u 
                $DOCKERHUB_CREDENTIALS_USR --password-stdin' 
                sh 'sudo docker push dhananjay/image'
            } 
        } 
        stage('Kubernetes') { 
            agent {  
                label 'KMaster' 
            }  
            steps { 
                sh 'kubectl create -f deploy.yml' 
                sh 'kubectl create -f svc.yml' 
            } 
        }                
    } 
} 
f. Go manage jenkins--> credentials->system---> global credentials-- >kind 
username and password -> provide username and the password. of DOCKERHUB because in the start of pipeline it asks for Dockerhub credentials so we add credentials of Dockerhub in Globat credentials --> it provides us a ID for the same (5e9bbc62-6d6d-4399-b90d-7cf7ffbb3158) which we add in the start of pipeline 
   
Adding DockerHub credentials in a pipeline is necessary when your pipeline requires access to private DockerHub repositories or when it needs to push Docker images to DockerHub.
 
 


26. Push the code and the pipeline will get triggered. 
a) first time pipeline did not triggered same happened when intellipat faculty was teaching us .
b) so manually build it and later it got triggered when Pushed the code .
 
 
Successful

