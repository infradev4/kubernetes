# TP-8: Créez un service de type nodeport

```txt
Ecrivez un manifest namespace.yml qui crée un namespace nommé production et lancez la création de ce namespace à partir du manifest
```

## Création d'un nouvel espace de noms

=> vi my-namespace.yml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

### Puis lancez :
=> kubectl create -f my-namespace.yml

```txt
Toutes vos prochaines ressources doivent être créées dans le namespace production
```
### Alternativement, vous pouvez créer un espace de noms à l'aide de la commande ci-dessous :

=> kubectl create namespace production

```txt
Ecrivez un manifest pod-red.yml pour déployer un pod avec l’image kodekloud/webapp-color en précisant que la color souhaitée est la rouge (red), le
pod doit posseder le label « app: web »
```

=> vi pod-red.yml 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webappred
  labels:
    app: web
    namespace: production
spec:
  containers:
  - name: webappred
    image: kodekloud/webapp-color
    ports:
    - containerPort: 8080
    env:
    - name: "APP_COLOR"
      value: red
```

```txt
Ecrivez un manifest pod-blue.yml pour déployer un pod avec l’image kodekloud/webapp-color en précisant que la color souhaitée est la bleue (blue) le
pod doit posseder le label « app: web »
```

=> vi pod-blue.yml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webappblue
  labels:
    app: web
    namespace: production
spec:
  containers:
  - name: webappblue
    image: kodekloud/webapp-color
    ports:
    - containerPort: 8080
    env:
    - name: "APP_COLOR"
      value: blue
```

```txt
Lancez la création des deux pods
```

=> kubectl create -f pod-red.yml

=> kubectl create -f pod-blue.yml

```txt
Ecrivez un manifest service-nodeport-web.yml qui permettra exposer les pods via un service de type node port, le nodeport devra être le 30008 et les target les ports 8080 de nos pods dont le label est « app: web »
```

=> vi service-nodeport-web.yml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  ports:
    #port du pod a utiliser
    - targetPort: 8080
    #port du service
      port: 8080
    #port utilisable depuis l'extrieur du cluster
      nodePort: 30008
  selector:
    app: web
```

```txt
Lancez la création du service et vérifiez qu’il trouve les deux pods (champ endpoint en utilisant la commande kubectl describe)
```

=> kubectl create -f service-nodeport-web.yml

http://54.152.199.15:30008/


