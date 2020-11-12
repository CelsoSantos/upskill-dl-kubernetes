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
export NAMESPACE="demo"
export K8S_USER="demo-user"
kubectl -n ${NAMESPACE} describe secret $(kubectl -n ${NAMESPACE} get secret | (grep ${K8S_USER} || echo "$_") | awk '{print $1}') | grep token: | awk '{print $2}'\n
```

5. 

```bash
kubectl  -n ${NAMESPACE} get secret `kubectl -n ${NAMESPACE} get secret | (grep ${K8S_USER} || echo "$_") | awk '{print $1}'` -o "jsonpath={.data['ca\.crt']}"
```

6. 

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJXRENCL3FBREFnRUNBZ0VBTUFvR0NDcUdTTTQ5QkFNQ01DTXhJVEFmQmdOVkJBTU1HR3N6Y3kxelpYSjIKWlhJdFkyRkFNVFUzTmpNd01qVXdNakFlRncweE9URXlNVFF3TlRRNE1qSmFGdzB5T1RFeU1URXdOVFE0TWpKYQpNQ014SVRBZkJnTlZCQU1NR0dzemN5MXpaWEoyWlhJdFkyRkFNVFUzTmpNd01qVXdNakJaTUJNR0J5cUdTTTQ5CkFnRUdDQ3FHU000OUF3RUhBMElBQlBiSmdKQ2w0elZsNFlaQUJ4dThnbFk1c2hEeDYwaVd6cXY0RU96MS93K24KKzJpMG5heUxuRTF1MjZmT1ZkY3dlMFdjUFJCb2J0M2ViNnNtYWRKQUhBV2pJekFoTUE0R0ExVWREd0VCL3dRRQpBd0lDcERBUEJnTlZIUk1CQWY4RUJUQURBUUgvTUFvR0NDcUdTTTQ5QkFNQ0Ewa0FNRVlDSVFEczN4YTArK0E4CktVd0w2NUZyR25vWW9sTWNUei81QnoxSmNJc292VkxncUFJaEFJV0xRUXpyWkpDem9TaVlEMFZvUXhlaXIwOUEKVWhnTDBucFlzRDVUUlVYKwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://k3s-master01:6443
  name: mycluster


contexts:
- context:
    cluster: mycluster
    namespace: demo
    user: demo-user
  name: demo

current-context: demo
kind: Config
preferences: {}


users:
- name: demo-user
  user:
    token:
    client-key-data:
```

7. `kubectl get secrets`
8. `kubectl get nodes`
9. `kubectl apply -f rbac/pods.yaml`

## NetworkPolicy

Source: https://docs.projectcalico.org/security/tutorials/kubernetes-policy-basic

1. `kubectl create ns policy-demo`
2. `kubectl create deployment --namespace=policy-demo nginx --image=nginx`
3. `kubectl expose --namespace=policy-demo deployment nginx --port=80`
4. `kubectl run --namespace=policy-demo access --rm -ti --image busybox /bin/sh`
5. `wget -q nginx -O -`
6. `exit`

Enable Isolation:
1. `kubectl apply -f network-policy/isolate.yaml`

Test Isolation:
1. `kubectl run --namespace=policy-demo access --rm -ti --image busybox /bin/sh`
2. `wget -q --timeout=5 nginx -O -`
3. `kubectl delete -f network-policy/isolate.yaml`

Allow Access:
1. `kubectl apply -f network-policy/allow.yaml`
2. `kubectl run --namespace=policy-demo access --rm -ti --image busybox /bin/sh`
3. `wget -q --timeout=5 nginx -O -`

However, we still cannot access the service from a pod without the label `run: access`. We can verify this as follows:
1. `kubectl run --namespace=policy-demo cant-access --rm -ti --image busybox /bin/sh`
2. `wget -q --timeout=5 nginx -O -`
3. `kubectl delete ns policy-demo`