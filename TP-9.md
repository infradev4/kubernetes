# Stockage persistent

## TP-9

```txt
Ecrivez un manifest mysql-volume.yml déployant un pod (nommé mysql-volume) mysql, avec les paramètre d’environnement
suivants : MYSQL_DATABASE: eazytraining, MYSQL_USER: eazy, MYSQL_PASSWORD: eazy, MYSQL_ROOT_PASSWORD: password
```

```yml
env:
      - name: MYSQL_DATABASE
        value: eazytraining
      - name: MYSQL_USER
        value: eazy
      - name: MYSQL_PASSWORD
        value: eazy
      - name: MYSQL_ROOT_PASSWORD
        value: password
```
```txt
Faites en sorte que le dossier contenant la base de données soit persistant en le montant sur votre nœud dans 
/data-volume en utilisant le principe de volumes

Par défaut MySQL écrir ses fichiers de données dans le répértoire /var/lib/mysql .
```
```yml
#point de montage au niveau du container
      volumeMounts:
      - mountPath: /var/lib/mysql
        name: mysql-data
    #creation du volume
  volumes:
   - name: mysql-data
     hostPath:
      path: /data-volume
      type: DirectoryOrCreate
```

## Obsérvation
### DirectoryOrCreate
```txt
Si rien n'existe au chemin fourni, un dossier vide y sera créé au besoin avec les permissions définies à 0755, avec le même groupe et la même possession que Kubelet.
```
=> vi mysql-volume.yml
```yml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
  labels:
    name: mysql
spec:
  containers:
    - name: mysql-volume
      image: mysql
      env:
      - name: MYSQL_DATABASE
        value: eazytraining
      - name: MYSQL_USER
        value: eazy
      - name: MYSQL_PASSWORD
        value: eazy
      - name: MYSQL_ROOT_PASSWORD
        value: password
  #point de montage au niveau du container
      volumeMounts:
      - mountPath: /var/lib/mysql
        name: mysql-data
    #creation du volume
  volumes:
   - name: mysql-data
     hostPath:
      path: /data-volume
      type: DirectoryOrCreate
```
```txt
Lancez la création de votre pod, vérifiez que votre pod a bien démarré et que ce dernier consomme effectivement le dossier local /data-volume
```
=> kubectl create -f mysql-volume.yml
```bash
pod/mysql-pod created
```

=>kubectl get pods
```bash
NAME        READY   STATUS    RESTARTS   AGE
mysql-pod   1/1     Running   0          3m26s
```

=> kubectl describe pods mysql-pod
```bash
 Environment:
      MYSQL_DATABASE:       eazytraining
      MYSQL_USER:           eazy
      MYSQL_PASSWORD:       eazy
      MYSQL_ROOT_PASSWORD:  password
    Mounts:
      /var/lib/mysql from mysql-data (rw)
```
```bash
Volumes:
  mysql-data:
    Type:          HostPath (bare host directory volume)
    Path:          /data-volume
    HostPathType:  DirectoryOrCreate
```

Obtenez un shell pour le conteneur en cours d'exécution:

kubectl exec -it mysql-pod -- /bin/bash

### Je vérifie la présence de "/data-volume" dans le POD
ubuntu@minikube-master:/data-volume$ => ls
```bash
bin   data-volume  etc  home  lib32  libx32      
```
### J'ajoute un fichier depuis le node:
ubuntu@minikube-master:/$ echo "galere" > /data-volume/galere.txt

### Je vérifie la présence de "galere.txt" dans le POD

root@mysql-pod:/# cat /var/lib/mysql/galere.txt
```bash
galere
```

```txt
Ecrivez pv.yml (volume persistent de taille 1 Go utilisant le dossier local /data-pv pour stocker les donneés) et pvc.yml (volume persistent claim de taille 100 Mo utilisant le PV créé précedement pour stocker les données)
```

## Créer le volume persistant

=> vi pv.yml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: "/data-pv"
```

=> kubectl apply -f pv.yml
```txt
persistentvolume/pv-volume created
```
## Afficher les informations sur le PersistentVolume :
=> kubectl get pv pv-volume
```sh
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-volume   1Gi        RWO            Retain           Available           manual                  28s
```
```txt
La sortie montre que le PersistentVolume a un STATUS de Available. Cela signifie qu'il n'a pas encore été lié à un PersistentVolumeClaim.
```

## Créer un PersistentVolumeClaim 
```txt
L'étape suivante consiste à créer un PersistentVolumeClaim. Les pods utilisent PersistentVolumeClaims pour demander un stockage physique.
```
=> vi pvc.yml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

### Créez le PersistentVolumeClaim : 

=> kubectl applique -f pvc.yml
```bash
persistentvolumeclaim/pv-claim created
```

### Regardez à nouveau le PersistentVolume : 

=> kubectl get pv pv-volume
```bash
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM              STORAGECLASS   REASON   AGE
pv-volume   1Gi        RWO            Retain           Bound    default/pv-claim   manual                  7m26s
```

La sortie montre que le PersistentVolumeClaim est lié à notre PersistentVolume.

## Créer un pod 

L'étape suivante consiste à créer un pod qui utilise le PersistentVolumeClaim comme volume. 

=> vi mysql-pv.yml
```yml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pv
  labels:
    name: mysql
spec:
  containers:
    - name: mysql-pv
      image: mysql
      env:
      - name: MYSQL_DATABASE
        value: eazytraining
      - name: MYSQL_USER
        value: eazy
      - name: MYSQL_PASSWORD
        value: eazy
      - name: MYSQL_ROOT_PASSWORD
        value: password
  #point de montage au niveau du container
      volumeMounts:
      - mountPath: /var/lib/mysql
        name: pv-storage
    #creation du volume
  volumes:
   - name: pv-storage
     persistentVolumeClaim:
      claimName: pv-claim
```

=>  kubectl apply -f mysql-pv.yml

=>  kubectl describe pods mysql-pv
```bash
 Environment:
      MYSQL_DATABASE:       eazytraining
      MYSQL_USER:           eazy
      MYSQL_PASSWORD:       eazy
      MYSQL_ROOT_PASSWORD:  password
    Mounts:
      /var/lib/mysql from pv-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7mghl (ro)

    Volumes:
  pv-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  pv-claim
```

Vérifiez que le conteneur dans le pod est en cours d'exécution ;

=>  kubectl get pod mysql-pv
```bash
NAME       READY   STATUS    RESTARTS   AGE
mysql-pv   1/1     Running   0          3m5s
```

ubuntu@minikube-master:~$ ls -lh /
```bash
total 92K
drwxr-xr-x   7 systemd-coredump root 4.0K Dec 20 22:17 data-pv
drwxr-xr-x   7 systemd-coredump root 4.0K Dec 20 21:17 data-volume
```

### Depuis mon node, j'écris dans le repertoire "/data-pv/"
ubuntu@minikube-master:~$ sudo sh -c "echo 'Bonjour depuis Kubernetes storage' > /data-pv/galere.txt"

### Depuis mon pode, je lis le fichier "galere.txt"
Obtenez un shell pour le conteneur en cours d'exécution dans votre pod :

=> kubectl exec -it mysql-pv -- /bin/bash
```bash
root@mysql-pv:/# cat /var/lib/mysql/galere.txt
Bonjour depuis Kubernetes storage
```

## Nettoyer 

### Supprimez le Pod, le PersistentVolumeClaim et le PersistentVolume :

```bash
kubectl delete pod mysql-pv
kubectl delete pvc pv-claim
kubectl delete pv pv-volume
```
