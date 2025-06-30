# minikube
### Initial Development On Minikube with Colima on MacOs
colima start --memory 8 --cpu 8
minikube start --driver=docker --memory=8192 --cpus=8

####minikube start --driver=docker --container-runtime=containerd --cpus=8 --memory=8g #works but microservice cart no start
##minikube start --driver qemu --network socket_vmnet --cpus=8 --memory=8g #works but istio-cni no start

# setup argocd
k create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
sleep 180
osascript -e 'tell application "Terminal" to do script "kubectl port-forward svc/argocd-server -n argocd 8080:443"'



kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
argocd login localhost:8080
argocd cluster add minikube --in-cluster
kubectl create secret generic github-ssh-key --from-file=sshPrivateKey=$HOME/.ssh/argocd_rsa -n argocd
argocd repo add git@github.com:shackdan/minikube.git --ssh-private-key-path ~/.ssh/argocd_rsa


## add metallb
minikube addon
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
kubectl apply -f ./manifests/metalLB/ip-pool.yaml

## test "external" access
### add istio gateway and virtualservice
kubectl apply -f ./manifest/sample-apps/the-app/gateway.yaml
kubectl apply -f ./manifest/sample-apps/the-app/virtualservice.yaml
### port forward
k port-forward svc/istio-ingressgateway -n istio-ingress 8081:80
### view traffic flow
istioctl dashboard kiali


### Troushooting
istioctl proxy-config routes istio-ingressgateway-6876cbbff7-nvkhw -n istio-ingress


###
CLEAN UP
Delete the argo
kubectl get applications.argoproj.io -n argocd -o name | xargs -n1 kubectl delete -n argocd

while true; do curl -I http://$GATEWAY_IP; sleep 0.5; done

while true; do curl -I http://10.97.69.155; sleep 0.5; done