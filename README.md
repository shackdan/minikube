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
### Initial Development On Minikube with Colima on MacOs
colima start --memory 8 --cpu 8
minikube start --driver=docker --memory=8192 --cpus=8
 

## Setup Aargocd

```
kubectl create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
sleep 180
export ARGOPASS=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
echo $ARGOPASS 
```

- Open new terminal window and run: ```argocd login localhost:8080``` - use 'admin' as user and the $ARGOPASS for the password

- Return to previous shell and continue:
```
argocd cluster add minikube --in-cluster
kubectl create secret generic github-ssh-key --from-file=sshPrivateKey=$HOME/.ssh/argocd_rsa -n argocd
argocd repo add git@github.com:shackdan/minikube.git --ssh-private-key-path ~/.ssh/argocd_rsa
```

## Add metallb
minikube addon enable metallb
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
kubectl apply -f ./manifests/metalLB/ip-pool.yaml

## Test "external" access
### Add istio gateway and virtualservice
kubectl apply -f ./manifest/sample-apps/the-app/gateway.yaml
kubectl apply -f ./manifest/sample-apps/the-app/virtualservice.yaml
### port forward
kubectl port-forward svc/istio-ingressgateway -n istio-ingress 8081:80

### view traffic flow
istioctl dashboard kiali


### Troushooting
istioctl proxy-config routes [istio-ingressgateway] -n istio-ingress


###
CLEAN UP
Delete the argo
kubectl get applications.argoproj.io -n argocd -o name | xargs -n1 kubectl delete -n argocd

while true; do curl -I http://$GATEWAY_IP; sleep 0.5; done

while true; do curl -I http://10.97.69.155; sleep 0.5; done