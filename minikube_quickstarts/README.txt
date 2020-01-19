MINIKUBE
minikube start --vm-driver=virtualbox --memory=10000 --cpus=6





##### FIRST TIME SETUP ######

NGINX INGRESS CONTROLLER CONTAINER / DEFAULT INGRESS TARGET
# this deploys a nginx pod which monitors /ingress on the api to get its routes
# it then generates an nginx config and uses that to route requests.
minikube addons enable ingress

this will start nginx from now on as a container whenever you start the cluster.  It will monitor the ingress API and do routing for you.
it also starts a default fallback for nginx to route to.
kubectl get pods -A | grep nginx
kubectl get pods -A | grep default


##### WHEN YOU CREATE OR RECREATE THE CLUSTER #####
Install HELM
kubectl apply -f rbac.yml
helm init --upgrade --service-account=tiller


this takes a few seconds to come up, nothing can find tiller till its up
kubectl get pods -A | grep tiller

##### EVERY TIME YOU START MINIKUBE ######

Setup DNS
get ip
minikube ip

put in etc hosts
192.168.99.119      k8s.local k8s.dashboard k8s.spinnaker k8s.gocd k8s.grafana


##### TOYS ######

DASHBOARD
helm install stable/kubernetes-dashboard --name kubernetes-dashboard -f dashboard.yml
navigate to k8s.dashboard

PROMETHEUS
helm install --name prometheus stable/prometheus -f prometheus.yml

GRAFANA
helm install --name grafana stable/grafana -f grafana.yml
admin/admin

JENKINS
helm install --name jenkins stable/jenkins -f jenkins.yml
navigate to k8s.local/jenkins
admin/admin

DRONE

helm install --name my-release stable/drone -f drone.yml
navigate to k8s.local/drone



GOCD

helm install --name gocd stable/gocd -f gocd.yml
navigate to k8s.gocd



SPINNAKER

helm install --name spinnaker stable/spinnaker --timeout 600 -f spinnaker.yml
navigate to k8s.spinnaker



ARGO

kubectl create namespace argo
kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=default:default
kubectl apply -n argo -f argo.yml          # pay attention to -n....
kubectl port-forward -n argo svc/argo-ui 9999:80

navigate to localhost:9999


cli is here:
https://github.com/argoproj/argo/releases

argo list
argo get xxx-workflow-name-xxx
argo logs xxx-pod-name-xxx #from get command above
argo submit --watch argoworkflows/hello-world.yaml
argo submit --watch argoworkflows/coinflip.yaml
argo submit --watch argoworkflows/loops-maps.yaml



ARGOCD


kubectl create namespace argocd
kubectl apply -n argocd -f argocd.yml

cli is here:
https://github.com/argoproj/argo-cd/releases/tag/v1.3.0

get password:
kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2

kubectl port-forward -n argocd svc/argocd-server 9998:80

login with cli:
argocd login <ARGOCD_SERVER>

coryleeio@M05t35:~/Workspace/k8sdev$ argocd app list
NAME  CLUSTER  NAMESPACE  PROJECT  STATUS  HEALTH  SYNCPOLICY  CONDITIONS  REPO  PATH  TARGET


or login at localhost:9998


argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated






add a remote helm chart

project default
cluster in cluster
namespace default
repo url
https://kubernetes-charts.storage.googleapis.com

chart mysql
version 1.6.0

enable automated
enable prune
enable self heal








EXTRAS

helm upgrade jenkins stable/jenkins -f jenkins.yml
helm upgrade prometheus stable/prometheus -f prometheus.yml
helm upgrade grafana stable/grafana -f grafana.yml
kubectl -n default port-forward svc/kubernetes-dashboard 8443:443
kubectl -n default port-forward svc/jenkins 8080:8080
minikube service jenkins --url

