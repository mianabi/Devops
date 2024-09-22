### Create user and it's Certs that can access to your K8S cluster

Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization

RBAC authorization uses the rbac.authorization.k8s.io API group to drive authorization decisions

```yml
useradd majid && cd /home/majid

openssl genrsa -out majid.key 2048

openssl req -new -key majid.key \
  -out majid.csr \
  -subj "/CN=majid"

openssl x509 -req -in majid.csr \
  -CA /root/.minikube/ca.crt \
  -CAkey /root/.minikube/ca.key \
  -CAcreateserial \
  -out majid.crt -days 500

mkdir .certs && mv majid.crt majid.key .certs

```
### and then:

```yml

kubectl config set-credentials majid \
  --client-certificate=/home/majid/.certs/majid.crt \
  --client-key=/home/majid/.certs/majid.key

kubectl config set-context majid-context \
  --cluster=kubernetes --user=majid

chown -R majid: /home/majid/

chmod 705 root 
```
### create role.yaml and rolebinding.yaml and run them:
```yml
vim role.yaml
vim rolebinding.yaml

kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml 
```
### make .kube/config file and copy on the path of your kuber path and test Authentication:
```yml
su majid
mkdir .kube && vi .kube/config

kubectl get po # Success
kubectl get role # No Access
```
