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


*Argo Rollouts *

1. Setting up argo rollouts
   
```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

2.install kubectl rollous plugin:

```bash
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64
sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts

kubectl argo rollouts version
```
3. Create the rollout and service

rollout: "kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/rollout.yaml"
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: rollouts-demo
spec:
  replicas: 5
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {}
      - setWeight: 40
      - pause: {duration: 10}
      - setWeight: 60
      - pause: {duration: 10}
      - setWeight: 80
      - pause: {duration: 10}
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: rollouts-demo
  template:
    metadata:
      labels:
        app: rollouts-demo
    spec:
      containers:
      - name: rollouts-demo
        image: argoproj/rollouts-demo:blue
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        resources:
          requests:
            memory: 32Mi
            cpu: 5m
```

service: "kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/service.yaml"
```yaml
apiVersion: v1
kind: Service
metadata:
  name: rollouts-demo
spec:
  ports:
  - port: 80
    targetPort: http
    protocol: TCP
    name: http
  selector:
    app: rollouts-demo
```

summery:
```bash
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/rollout.yaml
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/service.yaml
```

4. Watching rollouts via cli
```bash
kubectl argo rollouts get rollout rollouts-demo --watch
```
5. Dashboard
```bash
kubectl argo rollouts dashboard
```

6. Rollout a new image
Change the image of the docker image you are using. Experiment with blue, yellow, green, red, etc.
```bash
kubectl argo rollouts set image rollouts-demo \
  rollouts-demo=argoproj/rollouts-demo:yellow
```

Once you set the image, use the methods in step 2 to watch the rollout. If you want to connect to the application the below command works in minikube - otherwise you need to setup an ingress.
```bash
minikube service --all -n default
```
*reference*
https://argoproj.github.io/argo-rollouts/getting-started/
