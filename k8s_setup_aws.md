#### Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance
1. install AWSCLI
   ```sh
    curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
    apt install unzip python
    unzip awscli-bundle.zip
    #sudo apt-get install unzip - if you dont have unzip in your system
    ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    ```

1. Install kubectl
   ```sh
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```
1. Create an IAM user/role  with Route53, EC2, IAM and S3 full access
1. Attach IAM role to ubuntu server

    #### Note: If you create IAM user with programmatic access then provide Access keys.
   ```sh
     aws configure
    ```
1. Install kops on ubuntu instance:
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
1. create an S3 bucket
   ```sh
    aws s3 mb s3://dev.k8s.redmonkey.in
   ```
1. Expose environment variable:
   ```sh
    export KOPS_STATE_STORE=s3://dev.k8s.redmonkey.in
   ```
1. Create sshkeys before creating cluster
   ```sh
    ssh-keygen
   ```
1. Create kubernetes cluster definitions on S3 bucket
   ```sh
    kops create cluster --cloud=aws --zones=us-east-1a --name=dev.k8s.redmonkey.in --dns-zone=redmonkey.in --dns private
    ```
1. Create kubernetes cluser
    ```sh
      kops update cluster dev.k8s.redmonkey.in --yes
     ```
1. Validate your cluster
     ```sh
      kops validate cluster
    ```

1. To list nodes
   ```sh
     kubectl get nodes
   ```
#### Create Metrics server
1. Clone the github repository of metrics-server and move into metrics-server directory and create metrics-server
   ```sh
     git clone https://github.com/kubernetes-incubator/metrics-server.git
     cd metrics-server
     kubectl create -f deploy/1.8+/
     ```

1. now we can see metrics-server runing 
   ```sh
     kubectl get pods -n kube-system
   ```

#### Deploying tarunbhardwaj/flask-counter-app container on Kubernetes
1. Deploying tarunbhardwaj/flask-counter-app Container
    ```sh
      kubectl create -f deployment.yaml
      kubectl get pods
      kubectl get deployments
   ```

1. Expose the deployment as service. This will create an ELB in front of that 1 containers and allow us to publicly access them:
   ```sh
    kubectl create -f service.yaml
    kubectl get services -o wide
    ```

1. Create Horizontal Pod Autoscaler
  ```sh
  kubectl create -f hpa.yaml 
  kubectl get hpa
  ```

1. Get the public url of ELB in front of that 1 containers
  ```sh
   kubectl get services
  ``` 




 1. To delete cluster
    ```sh
     kops delete cluster dev.k8s.redmonkey.in --yes
    ```
