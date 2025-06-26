# EKS-Ingress-Alb-Deployment
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
    

kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
aws eks update-kubeconfig --name demo-cluster --region us-east-1
export AWS_REGION=us-east-1
export cluster_name=demo-cluster
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json ------ IAM POLICY 
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

    eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::324037294333:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve


# Download and install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify the installation
helm version
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks



helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  --namespace kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=vpc-0917934147da8a44e


kubectl get deployment -n kube-system aws-load-balancer-controller


===============================================================================================================================================================================================




🔹1. Set Up Environment & Cluster Config
Export required environment variables:
export AWS_REGION=us-east-1
export cluster_name=demo-cluster
export VPC_ID=vpc-0917934147da8a44e
Configure your kubeconfig to connect to the cluster:
aws eks update-kubeconfig --region $AWS_REGION --name $cluster_name
🔹 2. Create Fargate Profile for App Namespace
Create a Fargate profile to run pods in game-2048 namespace:
eksctl create fargateprofile \
  --cluster $cluster_name \
  --region $AWS_REGION \
  --name alb-sample-app \
  --namespace game-2048
🔹 3. Deploy 2048 Sample App (K8s Resources)
Create and apply the following YAML files:

namespace.yaml: defines the game-2048 namespace
deployment.yaml: runs the 2048 app in 2 pods
service.yaml: exposes the app internally using NodePort
ingress.yaml: exposes the app externally using an ALB via Ingress

kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
🔹 4. Associate IAM OIDC Provider for IRSA
Required for assigning IAM roles to Kubernetes service accounts:
eksctl utils associate-iam-oidc-provider \
  --region $AWS_REGION \
  --cluster $cluster_name \
  --approve
🔹 5. Create IAM Policy for ALB Controller
Download official policy for AWS Load Balancer Controller:
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
Create the policy in IAM:
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
🔹 6. Create IAM Service Account (IRSA)
Use eksctl to create a service account with the IAM policy:
eksctl create iamserviceaccount \
  --cluster=$cluster_name \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::324037294333:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
🔹 7. Install Helm (if not already installed)
Install Helm:
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
Add and update EKS Helm chart repo:
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
🔹 8. Install AWS Load Balancer Controller using Helm
Use Helm to deploy the controller into kube-system:
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  --namespace kube-system \
  --set clusterName=$cluster_name \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=$AWS_REGION \
  --set vpcId=$VPC_ID
🔹 9. Verify Setup
Check if the controller was deployed successfully:
kubectl get deployment -n kube-system aws-load-balancer-controller
Check if the ALB was provisioned for your ingress:
kubectl get ingress -n game-2048
The ADDRESS field should show the DNS name of the ALB.


























