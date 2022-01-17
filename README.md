## AWS EKS
```bash
export CLUSTER_NAME=""


```

1.  EKS 
```bash
export CLUSTER_REGION="us-east-2"
aws eks --region $CLUSTER_REGION update-kubeconfig --name $CLUSTER_NAME
```
1. AKS

### Register the Cluster
1. Get the current context which should be the cluster to register
```bash
export KUBECONFIG_CONTEXT= $(kubectl config current-context) 
```

Register the Cluster

```bash
 gcloud container hub memberships register $CLUSTER_NAME \
   --context=KUBECONFIG_CONTEXT \
   --kubeconfig="~/.kube/config" \
   --enable-workload-identity \
   --has-private-issuer
   ```