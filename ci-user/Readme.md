# Creating a Ci User in Kubernetes
The following steps will guide you through creating a user in Kubernetes that will be used by the CI/CD pipeline to deploy applications with Azure DevOps.

## Step 1: Create a Service Account
Create a service account in the Kubernetes cluster. This account will be granted the necessary permissions to deploy applications. 

```bash
kubectl create serviceaccount ci-user 
```

### Create the service account token
After Kubernetes version 1.24, the service account token is not automatically created. You need to create it manually. 
Create the following secret to store the service account token. 
```yaml
    apiVersion: v1
    kind: Secret
    metadata:
        name: secret-sa-ci-user
        namespace: default
    annotations:
        kubernetes.io/service-account.name: ci-user
    type: kubernetes.io/service-account-token
```
Then just apply: 
```bash
kubectl apply -f secret-sa-ci-user.yaml
```

## Step 2: Create a Cluster Role
Create a cluster role that will be used to grant permissions to the service account. 

```bash
kubectl create clusterrole ci-user-role \
    --verb=get,list,create,delete,patch,update \
    --resource=pods,deployments,replicasets,services,configmaps,secrets 
```

## Step 3: Bind the Service Account to the Cluster Role
Now you have your service account and cluster role, you need to bind the two together. Notice we use a RoleBinding instead of a ClusterRoleBinding. This is because we only want to grant permissions to the `ci-user` service account in specific namespaces.  
```bash
kubectl create rolebinding ci-user-role-binding \
    --role=ci-user-role \
    --serviceaccount=default:ci-user
    --namespace=<namespace-to-grant-permissions>
```
You need to apply this role binding in every namespace you want to grant permissions to the `ci-user` service account.

## Step 4: Create a service connection in Azure DevOps
Now, we need to create a service connection in Azure DevOps to authenticate with the Kubernetes cluster.
1. Go to your Azure DevOps project and navigate to `Project Settings` > `Service connections` > `New service connection` > `Kubernetes`.
2. Fill in the required fields and select the `Service Account` option.
3. Enter the server URL.
You can get it by running the following command:
    ```bash
    kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}'
    ```
4. Enter the secret of the service account you created earlier. 
You can get it by running the following command:
    ```bash
    kubectl get secret secret-sa-ci-user -oyaml
    ```
    Paste everything in the field.
5. Give the service connection a name and save it.


