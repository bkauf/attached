## Attach a Cluster to GCP Anthos
```bash
export CLUSTER_NAME="education-eks-6zZPtpwc"
export MEMBERSHIP_NAME="eks-attached"


```

###  EKS 
```bash
export AWS_REGION="us-east-2"
aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
```

#### Register the Cluster
1. Get the current context which should be the cluster to register
```bash
export KUBECONFIG_CONTEXT=$(kubectl config current-context) 
export OIDC_URL=$(aws eks describe-cluster --name $CLUSTER_NAME --region $AWS_REGION --query "cluster.identity.oidc.issuer" --output text)
```

Register the Cluster

```bash
 gcloud container hub memberships register $MEMBERSHIP_NAME \
   --context=$KUBECONFIG_CONTEXT \
   --kubeconfig="~/.kube/config" \
   --enable-workload-identity \
  --public-issuer-url=$OIDC_URL
   ```

### AKS 
[OIDC Issuer Feature in preview](https://docs.microsoft.com/en-us/azure/aks/cluster-configuration#register-the-enableoidcissuerpreview-feature-flag)

```
export CLUSTER_RG=""
export CLUSTER_NAME=""
az aks get-credentials --resource-group $CLUSTER_RG --name $CLUSTER_NAME
```

1. Get the current context which should be the cluster to register
```bash
export KUBECONFIG_CONTEXT=$(kubectl config current-context) 
export OIDC_URL=$(az aks show -n $CLUSTER_NAME -g $CLUSTER_RG --query "oidcIssuerProfile.issuerUrl" -otsv)
```

2. 

Register the Cluster

```bash
 gcloud container hub memberships register aks-attached \
   --context=$KUBECONFIG_CONTEXT \
   --kubeconfig="~/.kube/config" \
   --enable-workload-identity \
  --public-issuer-url=$OIDC_URL
   ```


#### Connect to the cluster with the Connect Gateway(Optional)
Use the following gcloud command to generate the nessessary impersonation rule and clusterrolebindings to authorize a GCP user to connect through the connect gateway. Replace [example@example.com] with a list of emails comma seperated. 

```bash
gcloud alpha container hub memberships generate-gateway-rbac  \
--membership=eks-attached \
--role=clusterrole/cluster-admin \
--users=[example@example.com] \
--project=multi-cloud-334222 \
--kubeconfig="~/.kube/config" \
--context=$KUBECONFIG_CONTEXT\
--apply
```

Take the output yaml and apply it to the cluster. Then you can connect with a new kubeconfig generated by gcloud
```bash
gcloud container hub memberships get-credentials eks-attached
kubectl get nodes
```




#### Connect to the cluster with the Connect Gateway(Optional)
Use the following gcloud command to generate the nessessary impersonation rule and clusterrolebindings to authorize a GCP user to connect through the connect gateway. Edit the ADMIN_EMAILS variable with a list of emails comma seperated google accounts. 

```bash

export ADMIN_EMAILS="example@example.com"

gcloud alpha container hub memberships generate-gateway-rbac  \
--membership=$MEMBERSHIP_NAME \
--role=clusterrole/cluster-admin \
--users=$ADMIN_EMAILS \
--project=$PROJECT_ID \
--kubeconfig="~/.kube/config" \
--context=$KUBECONFIG_CONTEXT\
--apply
```

Take the output yaml and apply it to the cluster. Then you can connect with a new kubeconfig generated by gcloud
```bash
gcloud container hub memberships get-credentials $MEMBERSHIP_NAME
kubectl get nodes
```

### V2 EKS Install
```sh

export ADMIN_EMAILS="example@example.com"
export CLUSTER_NAME="education-eks-6zZPtpwc"
export MEMBERSHIP_NAME="eks-attached-v2"
export AWS_REGION="us-east-2"
export PROJECT_NUMBER="xxxxxx"
export KUBECONFIG_CONTEXT=$(kubectl config current-context) 
export OIDC_URL=$(aws eks describe-cluster --name $CLUSTER_NAME --region $AWS_REGION --query "cluster.identity.oidc.issuer" --output text)

aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME

gcloud alpha container attached clusters register $CLUSTER_NAME \
  --location=us-east4 \
  --fleet-project=PROJECT_NUMBER \
  --platform-version=1.22 \
  --distribution=eks \
  --issuer-url=$ISSUER_URL \
  --context=$KUBECONFIG_CONTEXT \
  --admin-users=$ADMIN_EMAILS
  
  
  [--kubeconfig=KUBECONFIG_PATH] \
  [--description=DESCRIPTION]
```
