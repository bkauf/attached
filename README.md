## Attach a Cluster to GCP Anthos

The following instructions are for [EKS and AKS Clusters](https://cloud.google.com/anthos/clusters/docs/multi-cloud/attached). For [non EKS/AKS installs](https://cloud.google.com/anthos/clusters/docs/multi-cloud/attached/previous-generation/how-to/attach-kubernetes-clusters#attach-aks-kind-openshift-and-other-clusters) refer to V1 installation instructions below. 

### Prerequisites 
a. Enter the following configuration details for your attached cluster. The MEMBERSHIP_NAME is the name that will show up in the GCP Console. The cluster name is what shows up in the EKS or AKS console. 
```bash
export MEMBERSHIP_NAME="attached-cluster" 
export CLUSTER_NAME=""
export ADMIN_EMAILS="example@example.com"
export GCP_PROJECT_NUMBER="xxxxxx"
export KUBECONFIG_PATH=".kube/config"
```
b. Choose the closest [GCP Region](https://cloud.google.com/anthos/clusters/docs/multi-cloud/attached/eks/reference/supported-regions) to your cluster where the multi-cloud API is available. The GCP Multi-Cloud API will be hosted in this region
```sh
export GCP_REGION=[enter location]
```
c. Now execute the following command to view avaiable Attached cluster versions in that region. 

```sh
gcloud container attached get-server-config  \
  --location=$GCP_REGION
  ```
 d. Enter the Attached cluster version that matches the K8s version of your  cluster
```sh
export PLATFORM_VERSION=""
```

###  EKS 
Login to your EKS cluster and get the OIDC config
```bash
export AWS_REGION="us-east-"
aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
export OIDC_URL=$(aws eks describe-cluster --name $CLUSTER_NAME --region $AWS_REGION --query "cluster.identity.oidc.issuer" --output text)

```

### AKS 
a. Supply your Resource Group and get the OIDC config.  [OIDC Issuer Feature in preview](https://docs.microsoft.com/en-us/azure/aks/cluster-configuration#register-the-enableoidcissuerpreview-feature-flag)
```
export CLUSTER_RG=""
az aks get-credentials --resource-group $CLUSTER_RG --name $CLUSTER_NAME
```
b. Get the current context which should be the cluster to register
```bash
export KUBECONFIG_CONTEXT=$(kubectl config current-context) 
```
Turn on OIDC issuer in cluster if it is not already there
```bash
az aks update -g $CLUSTER_RG -n $CLUSTER_NAME --enable-oidc-issuer 
```
Get the OIDC URL
```bash
export OIDC_URL=$(az aks show -n $CLUSTER_NAME -g $CLUSTER_RG --query "oidcIssuerProfile.issuerUrl" -otsv)
```

## Register the Cluster
a. Get the current context which should be the cluster to register
```bash
export KUBECONFIG_CONTEXT=$(kubectl config current-context) 
```

b. Choose your K8s distrovution(eks or aks for the Attached Cluster V2 product, instruction for V1 below)

```sh
export DISTROBUTION="eks"
```

c. Register the cluster
```sh
gcloud container attached clusters register $MEMBERSHIP_NAME \
  --location=$GCP_REGION \
  --fleet-project=$GCP_PROJECT_NUMBER \
  --platform-version=$PLATFORM_VERSION \
  --distribution=$DISTROBUTION \
  --issuer-url=$OIDC_URL \
  --context=$KUBECONFIG_CONTEXT \
  --admin-users=$ADMIN_EMAILS \
  --kubeconfig=$KUBECONFIG_PATH \
  --description="Attached Cluster"
```
d. Use the connect gateway to login to the cluster through Google Cloud

```bash
gcloud container hub memberships get-credentials $MEMBERSHIP_NAME
kubectl get nodes
```
## V1 Attached Cluster Version - *Register non EKS/AKS clusters*

a. For Non EKS/AKS clusters use the registration command below

```bash
 gcloud container hub memberships register $MEMBERSHIP_NAME \
   --context=$KUBECONFIG_CONTEXT \
   --kubeconfig="~/.kube/config" \
   --enable-workload-identity \
   -public-issuer-url=$OIDC_URL
```


b.  Connect to the cluster with the Connect Gateway( V1 Clusters only,)
Use the following gcloud command to generate the nessessary impersonation rule and clusterrolebindings to authorize a GCP user to connect through the connect gateway. Replace [example@example.com] with a list of emails comma seperated. 

```bash
export GCP_PROJECT_ID=""
export ADMIN_EMAILS="[example@example.com,"example2@example.com]"
export $MEMBERSHIP_NAME="Attached Cluster"

gcloud alpha container hub memberships generate-gateway-rbac  \
--membership=$MEMBERSHIP_NAME \
--role=clusterrole/cluster-admin \
--users=$ADMIN_EMAILS \
--project=$GCP_PROJECT_ID \
--kubeconfig="~/.kube/config" \
--context=$KUBECONFIG_CONTEXT\
--apply
```

c. Take the output yaml and apply it to the cluster. Then you can connect with a new kubeconfig generated by gcloud
```bash
gcloud container hub memberships get-credentials $MEMBERSHIP_NAME
kubectl get nodes
```
