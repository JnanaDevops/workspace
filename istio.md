*Run the below commands to setup **Istio** in your minikube cluster *

```bash
minikube start

#stage1: download istio

curl -L https://istio.io/downloadIstio | sh -
cd istio-1.27.2 && export PATH=$PWD/bin:$PATH

#install istio using 'demo' profile

istioctl install --set profile=demo -y
kubectl get pods -n istio-system

#this adds a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later

kubectl label namespace default istio-injection=enabled

```
