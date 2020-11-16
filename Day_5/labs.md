# Upskill Labs - Day 5

## Minikibe setup

For this session, we will need to activate certain flags on the `apiserver` as well as a couple `minikube` addons.
To do so, start minikube using the following commands:

```bash
minikube start --nodes 3 --network-plugin=cni --cni=calico --extra-config=apiserver.authorization-mode=RBAC --extra-config=apiserver.enable-admission-plugins="LimitRanger,ResourceQuota,NamespaceExists,NamespaceLifecycle,ServiceAccount,DefaultStorageClass,MutatingAdmissionWebhook"
minikube addons enable dashboard
```

## Metrics

1. `kubectl edit service -n kubernetes-dashboard kubernetes-dashboard`
2. Modify service to be of type `NodePort`
3. `minikube service -n kubernetes-dashboard kubernetes-dashboard --url`
4. Open the URL on the browser and explore
5. `minikube addons enable metrics-server`
6. `kubectl get pods -A`
7. Open the URL on the browser and explore again, now with metrics

## Probes

Source: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

### Liveness

1. `kubectl apply -f probes/exec-liveness.yaml`
2. `kubectl describe pod liveness-exec` (before 30 secs have passed)
3. `kubectl describe pod liveness-exec` (after 30 secs have passed)
4. `kubectl describe pod liveness-exec` (after 60 secs have passed)

### Readiness

1. `kubectl apply -f probes/tcp-liveness-readiness.yaml`
2. `kubectl describe pod goproxy`

## Resource Allocation: Request, Limits & Quotas

### Requests & Limits

Source: https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/

Assign Memory Resources to Containers and Pods

1. `kubectl get apiservices` (or `kubectl get apiservices | grep metrics`)
2. You should see the following output:

```bash
NAME
v1beta1.metrics.k8s.io
```

3. `kubectl create namespace mem-example`
4. `kubectl apply -f resources/requests-limits/memory-request-limit.yaml`
5. `kubectl get pod -n mem-example memory-demo -o yaml`
6. `kubectl top pod -n mem-example memory-demo`
7. `kubectl delete pod -n mem-example memory-demo`
8. `kubectl apply -f resources/requests-limits/memory-demo-excess.yaml`
9. `watch -n 2 'kubectl get pod -n mem-example memory-demo-2'`
10. `kubectl get pod -n mem-example memory-demo-2 -o yaml`
11. You should see an output similar to the following:

```yaml
lastState:
   terminated:
     containerID: docker://65183c1877aaec2e8427bc95609cc52677a454b56fcb24340dbd22917c23b10f
     exitCode: 137
     finishedAt: 2017-06-20T20:52:19Z
     reason: OOMKilled
     startedAt: null
```

12. `kubectl describe pod -n mem-example memory-demo-2`
13. `kubectl describe nodes`
14. `kubectl delete pod -n mem-example memory-demo-2`
15. `kubectl apply -f resources/requests-limits/memory-too-big-for-node.yaml`
16. `kubectl get pod -n mem-example memory-demo-3`
17. `kubectl describe pod -n mem-example memory-demo-3`
18. You should see the pod is always `Pending`:

```yaml
Events:
  ...  Reason            Message
       ------            -------
  ...  FailedScheduling  No nodes are available that match all of the following predicates:: Insufficient memory (3).
```

19. `kubectl delete namespace mem-example`

### Namespace Quotas

Source:

- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/
- https://kubernetes.io/docs/tasks/administer-cluster/quota-api-object/

1. `kubectl create namespace quota-mem-cpu-example`
2. `kubectl apply -f resources/quota/resource-quota.yaml`
3. `kubectl get resourcequota -n quota-mem-cpu-example mem-cpu-demo -o yaml`
4. `kubectl apply -f resources/quota/quota-mem-cpu-pod.yaml`
5. `kubectl get resourcequota -n quota-mem-cpu-example mem-cpu-demo -o yaml`
6. `kubectl apply -f resources/quota/quota-mem-cpu-pod-2.yaml`
7. Pod should not be created and error with a message like:

```bash
Error from server (Forbidden): error when creating "examples/admin/resource/quota-mem-cpu-pod-2.yaml":
pods "quota-mem-cpu-demo-2" is forbidden: exceeded quota: mem-cpu-demo,
requested: requests.memory=700Mi,used: requests.memory=600Mi, limited: requests.memory=1Gi
```

8. `kubectl delete namespace quota-mem-cpu-example`

### API Objects Quotas

1. `kubectl create namespace quota-object-example`
2. `kubectl create -f resources/quota/object-quota.yaml`
3. `kubectl get resourcequota -n quota-object-example object-quota-demo -o yaml`
4. `kubectl create -f resources/quota/object-quota-pvc.yaml`
5. `kubectl get persistenvolumeclaims -n quota-object-example`
6. `kubectl create -f resources/quota/object-quota-pvc-2.yaml`
7. Should fail with a message similar to the following:

```bash
persistentvolumeclaims "pvc-quota-demo-2" is forbidden:
exceeded quota: object-quota-demo, requested: persistentvolumeclaims=1,
used: persistentvolumeclaims=1, limited: persistentvolumeclaims=1
```

8. `kubectl delete namespace quota-object-example`

## HorizontalPodAutoscaler

Source: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

1. `kubectl create ns autoscaler`
2. `kubectl apply -f hpa/php-apache.yaml`
3. `kubectl get pods -n autoscaler`
4. `kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10 -n autoscaler`
5. `kubectl get hpa -n autoscaler`
6. Open another terminal/session
7. `kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done" -n autoscaler`
8. *Wait around 60 seconds*
9.  `watch -n 2 'kubectl get hpa -n autoscaler'`
10. `kubectl get deployment -n autoscaler php-apache`
11. Stop load from the other terminal/session by pressing `Ctrl + C`
12. *Wait around 60 seconds again*
13. `kubectl get hpa -n autoscaler`
14. `kubectl get deployment -n autoscaler php-apache`

## Helm

Source:

### Installing from Repository

1. Open the browser on `https://artifacthub.io/` and explore
2. Search for `Jenkins`
3. `helm repo add jenkins https://charts.jenkins.io`
4. `helm repo update`
5. `helm show values jenkins/jenkins`
6. `helm install jenkins jenkins/jenkins --version 2.16.0  --dry-run`
7. `helm install jenkins jenkins/jenkins --version 2.16.0`
8.  `watch -n2 'kubectl get pods'`
9.  `kubectl describe pod XXXX`
10. `helm upgrade jenkins jenkins/jenkins --version 2.17.1`
11. `helm uninstall jenkins`

### Our own Helm Chart

1. `cd helm`
2. `helm create mychart`
3. Explore created directory
4. Open `mychart/templates/service.yaml`, for example
5. `helm install --dry-run --debug my-chart ./mychart`
6. `helm install --dry-run --debug my-chart ./mychart --set service.internalPort=8080`
7. `helm install my-chart ./mychart --set service.type=NodePort`
8. `minikube service XXXX --url`
9. Open the URL in the browser
10. `helm uninstall my-chart`
11. Create two copies of `values.yaml` and place them in `helm/`
    1.  Name one `values.dev.yaml`
    2.  The other `values.prod.yaml`
12. Label nodes to act as different environments
    1.  `kubectl label node minikube-m02 env=dev`
    2.  `kubectl label node minikube-m03 env=prod`
13. Modify the charts to deploy to different nodes depending on the nodeAffinity selector
    1. On `values.dev.yaml` place the following in the `nodeSelector` values: `env=dev`
    2. On `values.prod.yaml` do the same but this time using `env=prod`
14. `watch -n2 'kubectl get pods -A -o wide'`
15. `helm install -f values.dev.yaml my-dev-chart ./mychart`
16. `helm install -f values.prod.yaml my-prod-chart ./mychart`