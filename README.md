## Kubernetes NGINX+ Wordpress + Postgres + Persistent Volumes
Remarks: 
- There are two implementations: Minikube on Windows10 and Microk8s on Ubuntu 20.04
- Starting with MYSQL, wordpress is needed to change the internal MYSQL requests  to POSTGRESS ones by plugin.

### Installation:
1) Install MicroK8s cluster  
```
$ sudo snap install microk8s --classic
$ sudo usermod -a -G microk8s $USER
$ sudo chown -f -R $USER ~/.kube
$ microk8s status --wait-ready
$ microk8s enable dashboard dns ingress
```
OR for minikube 
```
msiexec minikube-installer.exe
msiexec kubectl.exe
minikube start --memory=4096 --vm-driver=virtualbox
minikube status
kubectl cluster-info
minikube addons enable ingress
```

2) Generate certs manually
```
$ sudo apt install -yq certbot
$ sudo certbot certonly --standalone -d it-tank.ru
$ sudo cp /etc/letsencrypt/live/it-tank.ru/cert.pem /home/tank/aspose/certs/tls-cert.crt
$ sudo cp /etc/letsencrypt/live/it-tank.ru/privkey.pem /home/tank/aspose/certs/tls-key.key
```

3)  Create k8s secret for certs located in ./certs
```
$ cd ~/tank
$ microk8s kubectl create secret tls it-tank --key ./certs/tls-key.key --cert ./certs/tls-cert.crt -n tank
$ microk8s kubectl label secret it-tank app=nginx -n tank
```

4) Create Resources (Services, Ingress, Deployments ...)
```
$ microk8s kubectl apply -f postgres.yaml 
$ microk8s kubectl apply -f mysql.yaml 
$ microk8s kubectl apply -f nginx-wp.yaml
$ microk8s kubectl apply -f micro-ingress.yaml  
```
OR for minikube 
```
kubectl apply -f postgres.yaml 
kubectl apply -f mysql.yaml 
kubectl apply -f nginx-wp.yaml
kubectl apply -f minikube-ingress.yaml 
```

5) Correct an identification plugin for DB  'root' account 
```
$ kubectl exec -it -n tank MYSQL_POD_NAME sh
# mysql -p
mysql> ALTER USER root IDENTIFIED WITH mysql_native_password BY 'rootroot';
```

6) Access https://it-tank.ru
Demo Wordpress Credentials:
- username: admin
- password: 123123123

### Changing DB


### Deletion
```
$  kubectl delete svc,pv,pvc,deployments,pods,secrets,ingress  -l app=nginx -n tank
$ kubectl delete svc,pv,pvc,deployments,pods,secrets,ingress  -l app=mysql -n tank
$ kubectl delete svc,pv,pvc,deployments,pods,secrets,ingress  -l app=wordpress -n tank
$ kubectl delete ns tank
```

