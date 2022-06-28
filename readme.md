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

### Acess argoCD UI

To access The ArgoCD UI, you need to:

`kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'`

OR

`kubectl port-forward -n argocd svc/argocd-server 8080:443`

### get the password of the admin user (the default user)

The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace. You can simply retrieve this password using kubectl:

`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo`

## 2. Configure ArgoCD with `Application` CRD

Let's write a configuration file for argoCD to connect it to the git repository where the config files are hosted

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  #Every application belongs to a signle project and you can group multiple application into a project
  project: default

  # Git Repository that argocd will connect to and sync it
  source:
    repoURL: https://github.com/mahdibouaziz/argoCD-demo.git
    targetRevision: HEAD
    path: dev # The folder in the repository

  # K8s Cluster where argocd will apply the manifest files in the git repository
  destination:
    server: https://kubernetes.default.svc # the default service name of K8s
    namespace: myapp

  syncPolicy:
    syncOptions:
      - CreateNamespace=true #to tell K8s to automatically create a namespace if it doesn't exists

    automated: #enable automated sync (self healing & pruning)
      selfHeal: true
      prune: true
```

## 3. Test our setup by updating Deployment.yaml file

we need now to apply this configuration to configure argoCD with this logic

`kubectl apply -f application.yaml`

this is gonna be the only kubectl apply that we need to do in this project, because after that everything shoud be auto synchronized

If we go to the argoCD UI we will see:
![Alt text](./images/argocd-ui.png?raw=true)
![Alt text](./images/argocd-ui2.png?raw=true)

And you can see the pods are created
![Alt text](./images/demo.png?raw=true)

## 4. Test automatic sync

update the image version for example in the deployment file to 1.1 or 1.2. Commit and push your updates and after a certain period of time you will see the update.
