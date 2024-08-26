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

kubectl config set-credentials majid \
  --client-certificate=/home/majid/.certs/majid.crt \
  --client-key=/home/majid/.certs/majid.key

kubectl config set-context majid-context \
  --cluster=kubernetes --user=majid

chown -R majid: /home/majid/

chmod 705 root 

vim role.yaml
:
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader-user
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

vim rolebinding.yaml
:
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: user-read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: majid # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader-user # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io

kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml 

su majid
mkdir .kube && vi .kube/config
:
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /root/.minikube/ca.crt
    server: https://192.168.49.2:8443
  name: my-cluster
contexts:
- context:
  name: default-context
  context:
    cluster: my-cluster
    user: majid
  name: majid-context
current-context: majid-context
preferences: {}
users:
- name: majid
  user:
    client-certificate: /home/majid/.certs/majid.cert
    client-key: /home/majid/.certs/majid.key

kubectl get po # Success
kubectl get role # No Access
