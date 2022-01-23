## Attach a Cluster to GCP Anthos
```bash
export CLUSTER_NAME="education-eks-6zZPtpwc"


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
 gcloud container hub memberships register eks-attached \
   --context=$KUBECONFIG_CONTEXT \
   --kubeconfig="~/.kube/config" \
   --enable-workload-identity \
  --public-issuer-url=$OIDC_URL
   ```
