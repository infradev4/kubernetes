# TP-6: Déployez votre première application

## Enoncé
```txt
Ecrivez un manifest pod.yml pour déployer un pod avec l’image kodekloud/webapp-color en précisant que la color souhaitée est la rouge.
```
vi pod.yml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color-red
  labels:
    app: webapp
    env: prod
spec:
  containers:
  - name: webappred
    image: kodekloud/webapp-color
    ports:
    - containerPort: 8080
    env:
    - name: TEST
      value: Frazer
```
```txt
Lancez votre pod et vérifiez qu’il est bien en cours d’exécution
```
kubectl create -f pod.yml

```txt
Exposez votre pod en utilisant la commande kubectl port-forward webapp-color-red 8080:8080 –-address 0.0.0.0
```

kubectl port-forward webapp-color-red 8080:8080 --address 0.0.0.0

http://54.152.199.15:8080/

```txt
Ecrivez un manifest nginx-deployment.yml pour déployer 2 replicas d’un pod nginx (en 1.21.3)
```

vi nginx-deployment.yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dp-nginx
  labels:
    role: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: web
      labels:
        app: nginx
        type: pod
        formateur: Frazer
    spec:
      containers:
        - name: web1
          image: nginx:1.21.3
          ports:
            - containerPort: 8080
```
```txt
Lancez le deployment, vérifier le nombre de pods et vérifiez que le deployment et le replicaset (ainsi que la version de l’image utilisée) ont été créé
```

kubectl create -f nginx-deployment.yml
kubectl get pod

```bash
NAME                       READY   STATUS    RESTARTS   AGE
dp-nginx-55fc99665-nkslr   1/1     Running   0          11s
dp-nginx-55fc99665-wwb5v   1/1     Running   0          11s
webapp-color-red           1/1     Running   0          7m36s
```

kubectl describe deployments.apps dp-nginx
```bash
Name:                   dp-nginx
Namespace:              default
CreationTimestamp:      Mon, 20 Dec 2021 18:38:23 +0000
Labels:                 role=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
           formateur=Frazer
           type=pod
  Containers:
   web1:
    Image:        nginx:1.21.3
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   dp-nginx-55fc99665 (2/2 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m36s  deployment-controller  Scaled up replica set dp-nginx-55fc99665 to 2

```
```txt
Modifiez le fichier nginx-deployment.yml afin d’utiliser l’image nginx en version latest, appliquer la modification
```

vi nginx-deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dp-nginx
  labels:
    role: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: web
      labels:
        app: nginx
        type: pod
        formateur: Frazer
    spec:
      containers:
        - name: web1
          image: nginx
          ports:
            - containerPort: 8080
```

```txt
Que se passe t’il ? Combien de replicasets avez-vous ? Quelle est l’image utilisée par le replicaset en cours d’utilisation ?
```

kubectl apply -f nginx-deployment.yml

```bash
ubuntu@minikube-master:~$ kubectl apply -f nginx-deployment.yml
Warning: resource deployments/dp-nginx is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/dp-nginx configured
```

vi nginx-deployment-latest.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dp-nginx
  labels:
    role: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: web
      labels:
        app: nginx
        type: pod
        formateur: Frazer
    spec:
      containers:
        - name: web1
          image: nginx
          ports:
            - containerPort: 8080
```

kubectl apply -f nginx-deployment-latest.yaml

kubectl get rs --show-labels

```sh
NAME                  DESIRED   CURRENT   READY   AGE     LABELS
dp-nginx-55fc99665    0         0         0       12m     app=nginx,formateur=Frazer,pod-template-hash=55fc99665,type=pod
dp-nginx-864d97bc7d   2         2         2       5m31s   app=nginx,formateur=Frazer,pod-template-hash=864d97bc7d,type=pod
```

kubectl get po --show-labels
```sh
NAME                        READY   STATUS    RESTARTS   AGE     LABELS
dp-nginx-864d97bc7d-2ndwh   1/1     Running   0          5m45s   app=nginx,formateur=Frazer,pod-template-hash=864d97bc7d,type=pod
dp-nginx-864d97bc7d-5glvp   1/1     Running   0          5m43s   app=nginx,formateur=Frazer,pod-template-hash=864d97bc7d,type=pod
webapp-color-red            1/1     Running   0          19m     app=webapp,env=prod
```
