# Building Istio/Argocd Lab
This repo is meant to serve as a guide to get a working K8s/Istio lab/demo environment.
Note: The repo was developed on a Apple Silicon MacBook
## Prep and Pre reqs
### CLI Tools
- brew: https://brew.sh
- minikube: https://minikube.sigs.k8s.io/docs/
- colima: https://github.com/abiosoft/colima
- kubectl: https://kubernetes.io/docs/reference/kubectl/
- argocd: https://argo-cd.readthedocs.io/

#### Install tools
- install brew: ```/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"```
- install colima: ```brew colima```
- install minikube: ```brew install minikube```
- install kubectl: ```brew install kubernetes-cli```
- install argocd: ```brew install argocd```
## Initial Development On Minikube with Colima on MacOs
colima start --memory 8 --cpu 8
minikube start --driver=docker --memory=8192 --cpus=8
 

## Setup Aargocd

```
kubectl create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl rollout status deployment/argocd-server -n argocd
export ARGOPASS=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
echo "Argocd Admin user password:\n ${ARGOPASS}"
```

- Open new terminal window and run: ```kubectl port-forward svc/argocd-server -n argocd 8080:443```

- Return to previous shell and continue - note password from previous step:
```
argocd login localhost:8080
argocd cluster add minikube --in-cluster
kubectl create secret generic github-ssh-key --from-file=sshPrivateKey=$HOME/.ssh/argocd_rsa -n argocd
argocd repo add git@github.com:shackdan/minikube.git --ssh-private-key-path ~/.ssh/argocd_rsa
```
Access Argo site via https://localhost:8080
Accept SSL security warning and continue to site.
Use Admin and password from previous step to login.
Note no applications are present.


## Add Apps
```
kubectl apply -f ./bootstrap/root-app.yaml
```
### Review Status


## Add metallb
minikube addon enable metallb
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
kubectl apply -f ./manifests/metalLB/ip-pool.yaml

## Test "external" access
### Add istio gateway and virtualservice
kubectl apply -f ./manifest/sample-apps/the-app/gateway.yaml
kubectl apply -f ./manife/sample-apps/the-app/virtualservice.yaml
### port forward
kubectl port-forward svc/istio-ingressgateway -n istio-ingress 8081:80

### view traffic flow
Create external traffic buy running curl from local shell

```while true; do curl -I http://10.97.69.155; sleep 0.5; done```

istioctl dashboard kiali


## Troushooting
### Check proxy routes
istioctl proxy-config routes [istio-ingressgateway] -n istio-ingress


## CLEAN UP
```minikube delete```

```colima stop```

while true; do curl -I http://$GATEWAY_IP; sleep 0.5; done

while true; do curl -I http://10.97.69.155; sleep 0.5; done