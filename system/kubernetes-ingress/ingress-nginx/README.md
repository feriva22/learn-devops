# How to Deploy Ingress Nginx In Kubernetes

## 1. Deploy with `helm` for latest stable
This option you must install helm CLI first in your computer, then execute this command\
`helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace`

## 2. Exposting service Nginx Ingress
The default for ingress nginx type service is `LoadBalancer`, just wait External-IP get\
If you have some trouble when getting External-IP because use Minikube not Cloud service, you can use command\
`minikube addons enable ingress`\
Then if you check its only accessed over Internal IP of Docker container , you can check it with command\
`minikube ip`\
That IP is your clusters IP Host Docker, if you want access with another IP in you HOST example IP Public, then you need to redirect request from IP public to your IP Host Docker\
`kubectl port-forward --namespace ingress-nginx --address 0.0.0.0 service/ingress-nginx-controller 80:80 &`\
`kubectl port-forward --namespace ingress-nginx --address 0.0.0.0 service/ingress-nginx-controller 443:443 &`\