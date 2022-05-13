## Kubernetes NGINX+ Wordpress + Postgres + Persistent Volumes
Remark: Starting with MYSQL, wordpress is needed to change the internal MYSQL requests  to POSTGRESS ones by plugin.

### Installation:
1) Install MicroK8s cluster  
```
$ sudo snap install microk8s --classic
$ sudo usermod -a -G microk8s $USER
$ sudo chown -f -R $USER ~/.kube
$ microk8s status --wait-ready
$ microk8s enable dashboard dns ingress

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
$ microk8s kubectl create secret tls tls-certificate --key ./certs/tls-key.key --cert ./certs/tls-cert.crt -n tank
$ microk8s kubectl label secret tls-certificate app=nginx -n tank
```

4) Create Resources (Services, Ingress, Deployments ...)
```
$ microk8s kubectl apply -f postgres.yaml 
$ microk8s kubectl apply -f mysql.yaml 
$ microk8s kubectl apply -f nginx-wp.yaml
$ microk8s kubectl apply -f ingress.yaml 
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

