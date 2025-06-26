# EKS-Ingress-Alb-Deployment
End to end project on EKS
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
aws eks update-kubeconfig --name demo-cluster --region us-east-1
