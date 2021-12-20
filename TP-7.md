# ConfigMap

## Information
```txt
Un ConfigMap est une ressource Kubernetes pour injecter la configuration dans les conteneurs. Ils nous permettent de conserver les paramètres de notre pile séparément de son code. 

Pour créer un ConfigMap, il existe plusieurs méthodes:
    Vous pouvez utiliser  kubectl create configmap
    un générateur ConfigMap dans kustomization.yaml pour créer un ConfigMap.
La méthode que je vais utiliser consiste à ajouter dans mon fichier de déploiement de pod les informations relatives au ConfigMap.
```
Énoncé:
## 
```txt
Créez à l’aide d’un manifest, un pod avec les caractéristiques suivantes :
• Nom : webapp-color
• Image: kodekloud/webapp-color
• Variable d’environnement: APP_COLOR = red (Avec configMap)
• containerPort: 8080; HostPort: 8080
```
## TP-7

```
vi tp7-configmap.yml
```
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: appcolor
data:
  color: red

---

apiVersion: v1
kind: Pod
metadata:
  name: webapp-color-configmap
  labels:
    app: webapp
    env: prod
    formateur:  Frazer
spec:
  containers:
    - name: webapp-color
      image: kodekloud/webapp-color
      ports:
        - containerPort: 8080
      env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:
            name: appcolor
            key:  color
```
## Appliquer la configuration

```
kubectl apply -f tp7-configmap.yml
```
### Écoute sur le port 8080 sur toutes les adresses, transfert vers 8080 dans le pod
```
kubectl port-forward webapp-color-configmap 8080:8080 --address 0.0.0.0
```
