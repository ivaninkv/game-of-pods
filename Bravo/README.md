# Bravo

## Exercise

### users
* Users connect to frontend service: `drupal-service`

### drupal-service
* frontend service name: `drupal-service`
* drupal-service configured as `NodePort`
* drupal-service uses NodePort `30095`

### drupal
* Deployment Name: `drupal`
* Replicas: `1`
* Image: `drupal:8.6`
* Deployment has an initContainer, name: `init-sites-volume`
* initContainer `init-sites-volume`, image: `drupal:8.6`
* initContainer `init-sites-volume`, persistentVolumeClaim: `drupal-pvc`
* initContainer `init-sites-volume`, mountPath: `/data`
* initContainer `init-sites-volume`, Command: `[ "/bin/bash", "-c" ]`, initContainer: Args: `[ 'cp -r /var/www/html/sites/ /data/; chown www-data:www-data /data/ -R' ]`
* Deployment `drupal` uses correct pvc: `drupal-pvc`
* Deployment has a regular container, name: `drupal`, image: `drupal:8.6`
* container: `drupal`, Volume mountPath: `/var/www/html/modules`, subPath: `modules`
* container: `drupal`, Volume mountPath: `/var/www/html/profiles`, subPath: `profiles`
* container: `drupal`, Volume mountPath: `/var/www/html/sites`, subPath: `sites`
* container: `drupal`, Volume mountPath: `/var/www/html/themes`, subPath: `themes`
* Deployment: `drupal` running
* Deployment: `drupal` has label `app=drupal`

### drupal-pvc
* Claim Name: `drupal-pvc`
* Storage Request: `5Gi`
* Access modes: `ReadWriteOnce`

### drupal-pv
* Access modes: `ReadWriteOnce`
* Volume Name: `drupal-pv`
* Storage: `5Gi`

### drupal-pv-hostpath
* Configure drupal-pv with `hostPath = /drupal-data` (create the directory on Worker Nodes)

### drupal-mysql-service
* Name: `drupal-mysql-service`
* Type: `ClusterIP`
* Port: `3306`

### drupal-mysql-secret
* Secret Name: `drupal-mysql-secret`
* Secret: `MYSQL_ROOT_PASSWORD=root_password`
* Secret: `MYSQL_DATABASE=drupal-database`

### drupal-mysql
* Name: `drupal-mysql`
* Replicas: `1`
* Image: `mysql:5.7`
* Deployment Volume uses PVC : `drupal-mysql-pvc`
* Volume Mount Path: `/var/lib/mysql`, subPath: `dbdata`
* Deployment: `drupal-mysql` running

### drupal-mysql-pvc
* Claim Name: `drupal-mysql-pvc`
* Storage Request: `5Gi`
* Access modes: `ReadWriteOnce`

### drupal-mysql-pv
* Volume Name: `drupal-mysql-pv`
* Storage: `5Gi`
* Access modes: `ReadWriteOnce`

### drupal-mysql-pv-hostpath
* Configure drupal-mysql-pv with `hostPath = /drupal-mysql-data` (create the directory on Worker Nodes)

---

## Solution

All `*.yml` file that needed, you can find in current directory.
For usefull we can use autocompletion:
```shell
echo 'source <(kubectl completion bash)' >>~/.bashrc
```


### drupal-mysql-pv-hostpath & drupal-pv-hostpath
```shell
kubectl get nodes
ssh node01
mkdir /drupal-mysql-data
mkdir /drupal-data
exit
```

### [drupal-pv](drupal-pv.yml)
```shell
cat > drupal-pv.yml
kubectl apply -f drupal-pv.yml
```

### [drupal-mysql-pv](drupal-mysql-pv.yml)
```shell
cat > drupal-mysql-pv.yml
kubectl apply -f drupal-mysql-pv.yml
```

### [drupal-pvc](drupal-pvc.yml)
```shell
cat > drupal-pvc.yml
kubectl apply -f drupal-pvc.yml
```

### [drupal-mysql-pvc](drupal-mysql-pvc.yml)
```shell
cat > drupal-mysql-pvc.yml
kubectl apply -f drupal-mysql-pvc.yml
```

### [drupal-mysql-secret](drupal-mysql-secret.yml)
```shell
kubectl create secret generic drupal-mysql-secret --from-literal=MYSQL_ROOT_PASSWORD=root_password --from-literal=MYSQL_DATABASE=drupal-database
```

### [drupal-mysql](drupal-mysql.yml)
First create template:
```shell
kubectl create deployment drupal-mysql --image=mysql:5.7 --dry-run=client -o yaml > drupal-mysql.yml
```

And then change it:
```shell
cat > drupal-mysql.yml
kubectl apply -f drupal-mysql.yml
```

### [drupal-mysql-service](drupal-mysql-service.yml)
```shell
kubectl expose deployment drupal-mysql --name=drupal-mysql-service --type=ClusterIP --port=3306
```

### [drupal](drupal.yml)
```shell
cat > drupal.yml
kubectl apply -f drupal.yml
```

### [drupal-service](drupal-service.yml)
```shell
kubectl create svc nodeport drupal-service --node-port=30095 --tcp=30095 --dry-run=client -o yaml > drupal-service.yml

# edit drupal-service.yml for change selector app=drupal
cat > drupal-service.yml
kubectl apply -f drupal-service.yml
```
