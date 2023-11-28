## Proof of Concept ArgoCD
`Objective:` Use the concept of using ArgoCD as a CD tool.  Willing prove the technical and conceptual viability and the ability to technically implement the idea.
`Documentation:` Documentation means a brief description of the concept and analysis of technical capabilities. It also demonstrates a short execution plan with a selected technology stack, considers technical scenarios, and shows metrics.

Using this example, we will get a simple and reliable starter set of tools and practices for deploying applications of any complexity. Essentially, we need nothing only [our repository](https://github.com/Dennyyyyyyy/go-demo-app) and some code to automate	the entire CI process from build to artifact.

For experimental purposes, we will choose the "one application one cluster" approach. For this purpose, so use a "single host kubernetes cluster" and select [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) as the Delivery and Deploy system for the test environment.

`Architecture`
![enter image description here](https://argo-cd.readthedocs.io/en/stable/assets/argocd_architecture.png)

`How it works` [link to official site](https://argo-cd.readthedocs.io/en/stable/#how-it-works "Permanent link")

`ArgoCD` follows the  ***GitOps*** pattern of using Git repositories as the source of truth for defining the desired application state. Kubernetes manifests can be specified in several ways:
-   [kustomize](https://kustomize.io/)  applications
-   [helm](https://helm.sh/)  charts
-   [jsonnet](https://jsonnet.org/)  files
-   Plain directory of YAML/json manifests
-   Any custom config management tool configured as a config management plugin

`ArgoCD` is a cubernetes controller that continuously monitors running applications and compares the current state with the desired state. As stated in the repository, a diplomat whose current state	deviates from the target state is considered ***Out of Sync***, and argocd informs and visualizes differences, providing an option for automatic or manual synchronization	the desired state.

1. So before you start installing ArgoCD, let's prepare a separate local cluster 
```sh
$ k3d cluster create argo
... 
INFO[0029] Cluster 'argo' created successfully!         
INFO[0029] You can now use it like this: kubectl cluster-info

$ kubectl cluster-info
Kubernetes control plane is running at https://0.0.0.0:43297
CoreDNS is running at https://0.0.0.0:43297/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://0.0.0.0:43297/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy

$ k version
$ k get all -A
```
2. ArgoCD can sometimes be installed with a helm, and we'll do that next time. For now, let's use the installation file from the official Argo`s repository. To begin with, we'll use it to create an Argos list with actions that will install the system by default. It's a good practice to check the contents of the file beforehand if you're not sure of its source. 
```sh
$ kubectl create namespace argocd
namespace/argocd created

$ k get ns
NAME              STATUS   AGE
kube-system       Active   65m
default           Active   65m
kube-public       Active   65m
kube-node-lease   Active   65m
argocd            Active   13s

$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

$ k get all -n argocd
 
$ k get pod -n argocd -w
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-redis-b5d6bf5f5-sqlgw                        1/1     Running   0          1m23s
argocd-notifications-controller-589b479947-fm4kp    1/1     Running   0          1m23s
argocd-applicationset-controller-6f9d6cfd58-9rvjf   1/1     Running   0          1m23s
argocd-dex-server-6df5d4f8c4-nd829                  1/1     Running   0          1m23s
argocd-application-controller-0                     1/1     Running   0          1m23s
argocd-server-7459448d56-xnrvf                      1/1     Running   0          1m23s
argocd-repo-server-7b75b45897-qsw9z                 1/1     Running   0          1m23s
```
3. After installing the system, we need to access the ArgoCD GUI. This can be done using node-port or loadbalancer, which can be configured for the ArgoCD server service. Let's use port-forward to access Argo using local port 8080. We will link to the argocd-server service, which is located in the argocd namespace, kubectl will automatically find the service and install a portmap from local port 8080 to remote port 443
```sh
$ k port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:443&
[2] 13528
Forwarding from 0.0.0.0:8080 -> 8080
Handling connection for 8080
```
ArgoCD works with https by default, so when you try to open localhost:8080, you will get a certificate error. So, in a productive system, you need to install certificates and configure these issues.

4. Now we need to get the password to log in to the system. Let's use the kubectl get secret command. Specify the name of the script, as well as the format of the output json path .data.password This will return the base64 encoded password. We use pipe base64 -D to get the password in plain text
```sh
k -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d;echo
lLMOQyfflxdQCjbM
```
5. After logging in, create the application using the GUI. From now on, applications configured in ArgoCD will be automatically set to be updated in Kubernetes.

![SCR-20231128-pqye.png](https://photos.onedrive.com/share/AF0857E0B7D951EB!58381?cid=AF0857E0B7D951EB&resId=AF0857E0B7D951EB!58381&authkey=!AE4IDU0IynFFfPM&ithint=photo&e=fFjZtn)
