# Upskill Labs - Day 3

## Clone

git clone https://github.com/CelsoSantos/upskill-dl-kubernetes.git

## Minikube Configuration

Let's start minikube with a CNI plugin (SDN) that supports `NetworkPolicy` and also enable `RBAC`, and create a 2 node configuration

`minikube start --network-plugin=cni --cni=calico --extra-config=apiserver.authorization-mode=RBAC`
`minikube node add`

## RBAC

Source: https://computingforgeeks.com/restrict-kubernetes-service-account-users-to-a-namespace-with-rbac/

1. `kubectl create namespace demo`
2. `kubectl apply -f rbac/serviceaccount.yaml`

Let’s create a role which will give created account complete access to namespace resources.
1. `kubectl api-versions| grep rbac`
2. `kubectl apply -f rbac/admin-role.yaml`
3. `kubectl get roles -n demo`

A role can also be created with limited access to resources in a namespace
1. `kubectl apply -f rbac/limited-role.yaml`

### Bind

1. `kubectl apply -f rbac/rolebinding.yaml`
2. `kubectl get rolebindings --namespace demo`
3. `kubectl describe sa demo-user -n demo`
4. 

```bash
# This is your token, to use on step 6 as YOUR_TOKEN
export NAMESPACE="demo"
export K8S_USER="demo-user"
kubectl -n ${NAMESPACE} describe secret $(kubectl -n ${NAMESPACE} get secret | (grep ${K8S_USER} || echo "$_") | awk '{print $1}') | grep token: | awk '{print $2}'\n
```

5. 

```bash
# This is your server certificate, to use on the next step as YOUR_CERT_DATA
kubectl  -n ${NAMESPACE} get secret `kubectl -n ${NAMESPACE} get secret | (grep ${K8S_USER} || echo "$_") | awk '{print $1}'` -o "jsonpath={.data['ca\.crt']}"
```

6. 

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: YOUR_CERT_DATA
    server: https://minikube
  name: policy-demo
contexts:
- context:
    cluster: policy-demo
    namespace: demo
    user: demo-user
  name: demo
current-context: demo
kind: Config
preferences: {}
users:
- name: demo-user
  user:
    token: YOUR_TOKEN
    client-key-data: YOUR_CERT_DATA
```

7. `kubectl get secrets`
8. `kubectl get nodes`
9. `kubectl apply -f rbac/pods.yaml`

## NetworkPolicy

Source: https://docs.projectcalico.org/security/tutorials/kubernetes-policy-basic

1. `kubectl create ns policy-demo`
2. `kubectl create deployment --namespace=policy-demo nginx --image=nginx`
3. `kubectl expose --namespace=policy-demo deployment nginx --port=80`
4. `kubectl run --namespace=policy-demo access --rm -ti --image yauritux/busybox-curl /bin/sh`
5. `curl -v http://nginx`
6. `exit`

Enable Isolation:
1. `kubectl apply -f network-policy/isolate.yaml`

Test Isolation:
1. `kubectl run --namespace=policy-demo access --rm -ti --image yauritux/busybox-curl /bin/sh`
2. `curl -v -m 5 http://nginx`
3. `kubectl delete -f network-policy/isolate.yaml`

Allow Access:
1. `kubectl apply -f network-policy/allow.yaml`
2. `kubectl run --namespace=policy-demo access --rm -ti --image yauritux/busybox-curl /bin/sh`
3. `curl -v -m 5 http://nginx`

However, we still cannot access the service from a pod without the label `run: access`. We can verify this as follows:
1. `kubectl run --namespace=policy-demo cant-access --rm -ti --image yauritux/busybox-curl /bin/sh`
2. `curl -v -m 5 http://nginx`
3. `kubectl delete ns policy-demo`