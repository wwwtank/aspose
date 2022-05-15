# Kubernetes NGINX+ Wordpress + Postgres + Persistent Volumes
Remarks:  Starting with MySQL, Wordpress is needed to change the internal MySQL requests  to Postgres ones by the special plug-in.

## Installation:
### 1) Install K8s cluster  Minikube based on VirtualBox
```
$ sudo apt install virtualbox
$ sudo curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64  && chmod +x minikube
$ sudo mkdir -p /usr/local/bin/
$ sudo install minikube /usr/local/bin/
$ minikube start --vm-driver=virtualbox
$ minikube status
$ kubectl cluster-info
$ minikube addons enable ingress 
```

### 2) Generate certs manually
```
$ sudo apt install -yq certbot
$ sudo certbot certonly --standalone -d it-tank.ru
$ sudo cp /etc/letsencrypt/live/it-tank.ru/cert.pem /home/tank/aspose/certs/tls-cert.crt
$ sudo cp /etc/letsencrypt/live/it-tank.ru/privkey.pem /home/tank/aspose/certs/tls-key.key
```

### 3)  Create k8s secret for certs located in ./certs
```
$ cd ~/tank
$ kubectl create ns tank
$ kubectl create secret tls ingress-tls --key ./certs/tls-key.key --cert ./certs/tls-cert.crt -n tank
$ kubectl label secret ingress-tls app=nginx -n tank
```

### 4) Create Resources (Services, Ingress, Deployments ...)
```
kubectl apply -f postgres.yaml 
kubectl apply -f mysql.yaml 
kubectl apply -f nginx-wp.yaml
kubectl apply -f ingress.yaml 
```

### 5) Check DB and correct an identification plugin for 'root' account 
```
$ kubectl exec -it -n tank MYSQL_POD_NAME sh
mysql -p
mysql> ALTER USER root IDENTIFIED WITH mysql_native_password BY 'rootroot';
```
and check Postgress
```
$ kubectl exec -it -n tank POSTGRESS_POD_NAME sh
su postgres
$ psql
\l
\q
```

### 6) Open https://it-tank.ru
Demo Wordpress Credentials:
- username: admin
- password: password

## Changing DB


## Deletion
```
$ kubectl -n tank delete svc,pv,pvc,deployments,pods,secrets,ingress --all  
$ kubectl delete ns tank
```

