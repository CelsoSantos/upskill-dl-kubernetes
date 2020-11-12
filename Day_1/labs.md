# Upskill Labs - Day 1

## API Resources

`kubectl api-resources`

## Running Pods, Logs and Specs

1. `kubectl explain pods`
2. `kubectl run nginx --image=nginx --dry-run=client`
3. `kubectl run nginx --image=nginx`
4. `kubectl describe pod nginx`
5. `kubectl get pod nginx -o yaml >> nginx.yaml`
6. `kubectl logs -f nginx`
7. `kubectl delete pod nginx`
8. `kubectl create -f nginx.yaml`
9. repeat 5

## Deployments

1. `kubectl create deployment --image nginx my-nginx`
2. `kubectl get deployment my-nginx`
3. `kubectl edit deployment my-nginx (edit replicas)`
4. `kubectl scale deployment --replicas 4 my-nginx`
5. `kubectl apply -f https://k8s.io/examples/application/deployment.yaml --record`

## Services

1. `kubectl expose deployment/my-nginx`
2. `kubectl describe service X`
3. `kubectl logs -f nginx`
4. `kubectl run -i -t busybox --image=yauritux/busybox-curl --restart=Never`
5. `curl (IP/SVC/Domain)`
6. `kubectl expose deployment my-nginx --port=80 --type=NodePort`