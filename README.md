# Deploying 2048Game on Amazon EKS

Deploying the 2048Game application on Amazon EKS, encompassing the creation of a Fargate cluster, necessary configurations, resource deployment, configuring an OIDC provider, setting up the AWS Load Balancer Controller add-on, and managing ingress.

## Prerequisites

Before deploying 2048Game on Amazon EKS, ensure that you have the following prerequisites installed:

kubectl
eksctl
AWS CLI


## Create Fargate Cluster in EKS using eksctl

```
eksctl create cluster --name 2048Game --region us-east-1 --fargate
```

## Update Kubeconfig

```
aws eks update-kubeconfig --name 2048Game --region us-east-1
```

## Create Fargate profile

```
eksctl create fargateprofile \
    --cluster 2048Game \
    --region us-east-1 \
    --name 2048Game-App \
    --namespace game-2048
```

## Deploy the Deployment, Service and Ingress

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

## Check Pods , Service and Ingress

```
kubectl get pods -n game-2048
```

```
kubectl get svc -n game-2048
```

```
kubectl get ingress -n game-2048
```

## OIDC provider

```
eksctl utils associate-iam-oidc-provider --cluster 2048Game --approve
```


## How to setup alb add on

Download IAM policy

```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

Create IAM Policy

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

Create IAM Role

```
eksctl create iamserviceaccount \
  --cluster=2048Game \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::129128298531:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

## Deploy ALB controller

Add helm repo

```
helm repo add eks https://aws.github.io/eks-charts
```

Update the repo

```
helm repo update eks
```

Install

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  --namespace kube-system \
  --set clusterName=2048Game \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=vpc-0b64da2cc3d4a71e0
```

Verify that the deployments are running.

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

```
kubectl get ingress -n game-2048
```

## Delete the cluster

```
eksctl delete cluster --name 2048Game --region us-east-1
```