# How to Running ArgoCD for Continous Deployment

## Getting Started
For installing argocd with default config, we can use tutorial from [this link](https://argo-cd.readthedocs.io/en/stable/getting_started/)

### 1. Install ArgoCD
`kubectl create namespace argocd`\
`kubectl apply -n argocd -f install.yaml`\


### 2. Install ArgoCD CLI for management CLI
`curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64`

### 3. Access ArgoCD API Server
For access ArgoCD API you have some options, for default when first deployment we not expose service to Host port
this is options for access ArgoCD API Server\
a. Change Service Type `LoadBalancer`\
For access argocd from public/internet, we can change `service/argocd-server` from type `ClusterIP` to `LoadBalancer`, then wait for external IP to access from public\
`kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'`\
b. Ingress\
We can start to deploy Ingress service like Nginx Ingress in kubernetes cluster, then register `service/argocd-server` to exposed via Nginx Ingress\
`kubectl apply -f ingress/nginx-ingress.yaml`\

c. Port Forwarding\
This is easy method for exposing argocd service to Host deployed argocd inside clusters, the downside of this options is You must make sure port exposed **not used**, and access only directly to IP address of Host\
`kubectl port-forward svc/argocd-server -n argocd 8080:443`\
After you success execute that command then access service at `https://localhost:8080`\

### 4. Get Password for login in argocd
Execute this command to login as admin and get password
`argocd admin initial-password -n argocd`

### 5. Login to Argocd CLI for used it
`argocd login <ARGOCD_SERVER>`\
ARGOCD_SERVER is location of service argocd-server port forwarded

## Start Deploying app
We can use this example guestbook application to deploy in Argocd\
`argocd app create guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path guestbook --dest-server https://kubernetes.default.svc --dest-namespace default`\

Then we can check it from ArgoCD UI for check if application succesfully deployed 

## Check with Temporary pod, without change service Type to LoadBalancer or NodePort
`kubectl run temp-pod --rm -it --image=busybox --restart=Never -- sh`\
Then check with use `wget` command
`wget --no-check-certificate -O - http://guestbook-ui`
 