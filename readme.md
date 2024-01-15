# Description
This repo holds the basic configuration to setup a prometheus on EKS

# Prerequiites
Before install the Prometheus you need to have:
- EKS installed
- kubectl installed and configured
- EKS addon to enable new PVC automatically

## How to enable auto PVC creatgion with EKS Addon
### Create service account to enable auto PVC from eks after 1.23
```bash
kubectl config use-context <MY_CLUSTER_ARN>
eksctl create iamserviceaccount \
  --region us-east-1 \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster <CLUSTER_NAME> \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name AmazonEKS_EBS_CSI_DriverRole
```
### Create addon to enable the driver to eks stage
eksctl create addon --name aws-ebs-csi-driver --cluster <CLUSTER_NAME> --service-account-role-arn arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):role/AmazonEKS_EBS_CSI_DriverRole --force

# Install Prometheus helm
```bash
kubectl config use-context <MY_CLUSTER_ARN>
kubectl create namespace prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

# Install/Upgrade Blackbox
```bash
kubectl config use-context <MY_CLUSTER_ARN>
helm install prometheus-blackbox-exporter prometheus-blackbox-exporter \
    --namespace prometheus \
    --values blackbox-config.yml
```

# Install Prometheus with desired specs
```bash
helm upgrade --install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.size="60Gi" \
    --set server.retention="30d" \
    --set server.retentionSize="53GB" \
    --set-file extraScrapeConfigs=blackbox.yml \
    --values values.yml
```


