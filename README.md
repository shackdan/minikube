# minikube
### Initial Development On Minikube with Colima on MacOs
colima start --memory 8 --cpu 8
minikube start --driver=docker --memory=8192 --cpus=8 --force

####minikube start --driver=docker --container-runtime=containerd --cpus=8 --memory=8g #works but microservice cart no start
##minikube start --driver qemu --network socket_vmnet --cpus=8 --memory=8g #works but istio-cni no start

# setup argocd
k create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
sleep 180
kubectl port-forward svc/argocd-server -n argocd 8080:443


kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
argocd login localhost:8080
argocd cluster add minikube --in-cluster
kubectl create secret generic github-ssh-key --from-file=sshPrivateKey=$HOME/.ssh/argocd_rsa -n argocd
argocd repo add git@github.com:shackdan/minikube.git --ssh-private-key-path ~/.ssh/argocd_rsa


###
CLEAN UP
Delete the argo
kubectl get applications.argoproj.io -n argocd -o name | xargs -n1 kubectl delete -n argocd

while true; do curl -I http://$GATEWAY_IP; sleep 0.5; done