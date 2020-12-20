# kubernetes
Kubernetes playground

kubectl create ns finance

# Create key
openssl genrsa -out john.key 2048

# Certificate signing request, focus on -subj. user 'john' organization 'finance'
openssl req -new -key john.key -out john.csr -subj "/CN=john/O=finance"

# create certificate. Need ca.crt and ca.key from kubemaster /etc/kubernetes/pki
openssl x509 -req -in john.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out john.crt -days 365


# get cluster details
kubectl config view

# set cluster with cluster ca.crt
kubectl --kubeconfig john.kubeconfig config set-cluster kubernetes --server https://172.42.42.100:6443 --certificate-authority=ca.crt

# set user/client john with client-certificate and client -key
kubectl --kubeconfig john.kubeconfig config set-credentials john --client-certificate /export/home/hiren/play/temp/john.crt --client-key /export/home/hiren/play/temp/john.key

# set user context
kubectl --kubeconfig john.kubeconfig config set-context john-kubernetes --cluster kubernetes --namespace finance --user john

# use context (current context)
kubectl --kubeconfig john.kubeconfig config use-context john-kubernetes

# view config
kubectl --kubeconfig john.kubeconfig config view


'''
$ kubectl --kubeconfig john.kubeconfig get ns
Error from server (Forbidden): namespaces is forbidden: User "john" cannot list resource "namespaces" in API group "" at the cluster scope

$ kubectl --kubeconfig john.kubeconfig get pods
Error from server (Forbidden): pods is forbidden: User "john" cannot list resource "pods" in API group "" in the namespace "finance"
'''

# Alternet method for creating kube config file
cp ~/.kube/config john.new.kubeconfig

# Edit john.new.kubeconfig
# Update user, name, current-context and users. Delete client-certificate-data and client-key-data values. Insert base64 decoded data as below
cat john.crt | base64 -w0
cat john.key | base64 -w0



# Create ROLE
kubectl create role --help
kubectl create role john-finance --verb=get,list --resource=pods --namespace finance

# Create ROLE Binding to User
kubectl create rolebinding john-finance-rolebinding --role john-finance -n finance --user john

# Create ROLE Binding to Group
kubectl create rolebinding finance-rolebinding --role john-finance -n finance --group=finance

# Cluster ROLEs  (https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
kubectl --kubeconfig john.kubeconfig get deployments.apps
