# Demo Setup:

in this demo we are going to install ArgoCD in a K8s cluster, configure it with `Application` CRD and then test our setup by updating the Deployment file

## Requirements:

- Clone this git repository: https://github.com/mahdibouaziz/argoCD-demo
- You need to have a K8s cluster

## 1. Install ArgoCD in a K8s cluster

Create a new namespace, argocd, where Argo CD services and application resources will live.

`kubectl create namespace argocd`

This will create a lot of Pods and Services

`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

### Must open a discussion here !!!

To access The ArgoCD UI, you need to ?:

`kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'`

OR

`kubectl port-forward -n argocd svc/argocd-server 8080:443`

### get the password of the admin user (the default user)

The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace. You can simply retrieve this password using kubectl:

`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo`

## 2. Configure ArgoCD with `Application` CRD

## 3. Test our setup by updating Deployment.yaml file
