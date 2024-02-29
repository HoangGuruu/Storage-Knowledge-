```sh
# STEP WITH EKS
aws eks --region us-east-1 update-kubeconfig --name hr-stag-eksdemo1-guru

kubectl get nodes

# Download IAM Policy
## Download latest
curl -o iam_policy_latest.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

# Create IAM Policy using policy downloaded 
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy_latest.json

  
  
eksctl create iamserviceaccount \
  --cluster=hr-stag-eksdemo1 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::736059458620:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve



helm repo add eks https://aws.github.io/eks-charts

# Update your local repo to make sure that you have the most recent charts.
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=hr-stag-eksdemo1-guru \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=vpc-0956ac8dbf3553ffc \
  --set image.repository=602401143452.dkr.ecr.us-east-1.amazonaws.com/amazon/aws-load-balancer-controller



kubectl -n kube-system get deployment aws-load-balancer-controller

kubectl apply -f ingressclass.yaml

# Verify IngressClass Resource
kubectl get ingressclass

# Describe IngressClass Resource
kubectl describe ingressclass my-aws-ingress-class

```
# EKS Storage with EBS - Elastic Block Store

## Step-01: Introduction
- Create IAM Policy for EBS
- Associate IAM Policy to Worker Node IAM Role
- Install EBS CSI Driver

## Step-02:  Create IAM policyy
- Go to Services -> IAM
- Create a Policy 
  - Select JSON tab and copy paste the below JSON
```json

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:AttachVolume",
        "ec2:CreateSnapshot",
        "ec2:CreateTags",
        "ec2:CreateVolume",
        "ec2:DeleteSnapshot",
        "ec2:DeleteTags",
        "ec2:DeleteVolume",
        "ec2:DescribeInstances",
        "ec2:DescribeSnapshots",
        "ec2:DescribeTags",
        "ec2:DescribeVolumes",
        "ec2:DetachVolume"
      ],
      "Resource": "*"
    }
  ]
}
```
  - Review the same in **Visual Editor** 
  - Click on **Review Policy**
  - **Name:** Amazon_EBS_CSI_Driver
  - **Description:** Policy for EC2 Instances to access Elastic Block Store
  - Click on **Create Policy**


## Step-04: Deploy Amazon EBS CSI Driver  
- Verify kubectl version, it should be 1.14 or later
```
kubectl version --client --short
```
- Deploy Amazon EBS CSI Driver

# Deploy EBS CSI Driver
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"

# Verify ebs-csi pods running
kubectl get pods -n kube-system

```sh

kubectl apply -f storageclass.yaml

kubectl get storageclass

kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
kubectl patch storageclass ebs-sc -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

kubectl get storageclass

```


## Step-01: Introduction
- **External DNS:** Used for Updating Route53 RecordSets from Kubernetes 
- We need to create IAM Policy, k8s Service Account & IAM Role and associate them together for external-dns pod to add or remove entries in AWS Route53 Hosted Zones. 
- Update External-DNS default manifest to support our needs
- Deploy & Verify logs

## Step-02: Create IAM Policy
- This IAM policy will allow external-dns pod to add, remove DNS entries (Record Sets in a Hosted Zone) in AWS Route53 service
- Go to Services -> IAM -> Policies -> Create Policy
  - Click on **JSON** Tab and copy paste below JSON
  - Click on **Visual editor** tab to validate
  - Click on **Review Policy**
  - **Name:** AllowExternalDNSUpdates 
  - **Description:** Allow access to Route53 Resources for ExternalDNS
  - Click on **Create Policy**  
  
```sh
eksctl create iamserviceaccount \
    --name external-dns \
    --namespace default \
    --cluster hr-stag-eksdemo1-guru \
    --attach-policy-arn arn:aws:iam::736059458620:policy/AllowExternalDNSUpdates \
    --approve \
    --override-existing-serviceaccounts
```    



# List IAM Service Accounts using eksctl
eksctl get iamserviceaccount --cluster hr-stag-eksdemo1-guru


```sh
kubectl apply -f external-dns.yaml -n jenkins
kubectl get all
```

# Install Jenkins

```sh
kubectl create ns jenkins
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
chart=jenkinsci/jenkins
helm install jenkins -n jenkins $chart
```

```sh

kubectl get pv -n jenkins
kubectl get pvc -n jenkins
kubectl get pod -n jenkins
kubectl get svc -n jenkins


kubectl apply -f jenkins-ingress.yaml -n jenkins
# certificate manager
kubectl get ingress -n jenkins

```

# Add domain name to Hosted Zone

```sh
Access : jenkins.hoangguruu.id.vn

jsonpath="{.data.jenkins-admin-password}"
secret=$(kubectl get secret jenkins -n jenkins -o jsonpath=$jsonpath)
echo $(echo $secret | base64 --decode)

# DGreKWHli99otzySye4ZKR
```

# Install ArgoCD

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl edit svc argocd-server
# Add to domain on hosted zone
# Access argocd.hoangguruu.id.vn
```
# Access jenkin create jobs
```sh
#jenkins
XJ55CvKDQER5kE0Hcdxlcb

# argocd 
Abq08DuHEtuZkSbp

# github
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
```

kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=hoangguruu --docker-password=01694427257Aa@ --docker-email=nthoangthcs@gmail.com -n prod

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



docker run -d --rm --name=agent1 -p 4444:22 \
-e "JENKINS_AGENT_SSH_PUBKEY=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQD2CapAui7Ieuv7HOnZpqagGGVYezEuJ57Kij4/SpdxUHwThkiaQCElw8xL8hgxf0Zvoois6OM9Ee+i71bPlyXx/MbrpyXRC/UF3VsvmcyRSfEEa+F6SR3Uo38GRqPjaf/cy/XaupDd8PlsYkqAjjqba8/OylJfNHxnnyMmB2DjYAflbVFIjQa/xP1zRs5MQHGKPxT5GsIITzHhTncE//G8etxMSyLgB9k+EZuliPluu3lPuNQBAEzJM3ldu4rN4nRNVOqKsU3MnrhKvs541NHdndHZCZ5W37/M/B8M6C/CpCcUFX4S4gLt7a1dYslhoJ2btFq24S7OpKjMCyIiemCwgPxIEn15F6q8mBVpRlfN9LTXAo6/BINiYiFVlZAX+Zyv7Gy5d7hGogAcjxFLBa2kocRa9dBJ6ata6YtF8C5Y80Lunyo9wufO+dFcZ4ndEGxPvYNg4NSSvIX7n5K1Z+0p5sXD7uNHkNCY3KbjhV1S3JIgEzsjx2nA5HRS9Ox9fs0= ubuntu@ip-172-31-2-165" \
jenkins/ssh-agent:alpine-jdk17
# Acess argocd
VvJ2viyVFQTxHuwQ


```sh
# Install docker 

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo chmod 666 /var/run/docker.sock

```
# argocd
TsxrvgcRCBgmCxHr

# jenkins
ZeA3kqaj6TQHf9jMI9kdsZ