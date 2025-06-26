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
