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
kubectl rollout status deployment/controller -n metallb
kubectl apply -f ./manifests/metalLB/ip-pool.yaml

## Test "external" access
### port forward
Open a separate shell and run:
```minikube tunnel``` 
Note: this will prompt for sudo access on local

### Deploy Obeervability Tools
Apply manifests
```
kubectl apply -f ./manifests/
kubectl rollout status deployment/kiali -n istio-system
```

### prep hosts file
```sudo vi /etc/hosts```

Add the appropriate hostname mapping to point to loopback/127.0.0.1:
```
cat /etc/hosts
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost
127.0.0.1       app2.thenewtonlab.com
127.0.0.1       app1.thenewtonlab.com
127.0.0.1       demo.thenewtonlab.com
127.0.0.1       demo2.thenewtonlab.com
127.0.0.1       helloworld.thenewtonlab.com
```

### view traffic flow
Create external traffic buy running curl from local shell

```while true; do curl -I http://demo2.thenewtonlab.com; sleep 0.5; done```

istioctl dashboard kiali


## Troushooting
### Check proxy routes
istioctl proxy-config routes $(kubectl get pods -n istio-ingress -o name) -n istio-ingress


## CLEAN UP
```minikube delete```

```colima stop```
curl -v -H "Host: demo2.thenewtonlab.com" http://<INGRESS_IP>

while true; do curl -I http://$GATEWAY_IP; sleep 0.5; done

while true; do curl -I http://fqdn-serivehost.com; sleep 0.5; done


kubectl get pod web-frontend-5978cf9cc6-tbdzc -n test -o jsonpath='{.status.containerStatuses[?(@.name=="web")].lastState.terminated.exitCode}'


kubectl run -it --rm debug --image=curlimages/curl -- sh
# Once inside the pod
curl -v -H "Host: demo2.thenewtonlab.com" http://istio-ingressgateway.istio-ingress.svc.cluster.local
# Or directly to the IP
curl -v -H "Host: demo2.thenewtonlab.com" http://192.168.49.240