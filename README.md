# Kubernetes NGINX+ Wordpress + Postgres + Persistent Volumes
Remarks:  Wordpress is needed to change the internal MySQL requests  to Postgres ones by the special plug-in. This action performs manualy in appropriate containers.

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
kubectl apply -f nginx-wp.yaml
kubectl apply -f ingress.yaml 
```

### 5) Check DB and make plugin action on BD
```
$ kubectl exec -it -n tank POSTGRESS_POD_NAME sh
 su - postgres
 psql
 create database wp;
 create user adminwp with password '123123';
 grant all privileges on database wp to adminwp;
 q 
```

### 6) Install plug-in on Wordpress
```
$ kubectl exec -it -n tank WORDPRESS_POD_NAME sh
 cd wp-content
 apt-get update 
 apt-get install -y git
 git clone https://github.com/kevinoid/postgresql-for-wordpress.git
 mv postgresql-for-wordpress/pg4wp Pg4wp
 cp Pg4wp/db.php db.php

//check
 grep DB_DRIVER db.php
//this will return the following
//define('DB_DRIVER', 'pgsql'); 
```

### 7) Open https://it-tank.ru
Demo Wordpress Credentials:
- username: adminwp
- password: 123123

## Deletion
```
$ kubectl -n tank delete svc,pv,pvc,deployments,pods,secrets,ingress --all  
$ kubectl delete ns tank
```
