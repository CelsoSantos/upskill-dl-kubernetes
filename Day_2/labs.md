# Upskill Labs - Day 2

## Clone

git clone https://github.com/CelsoSantos/upskill-dl-kubernetes.git

## Ingress on Minikube

1. `minikube addons enable ingress`
2. `kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0`
3. `kubectl apply -f 01-ingress.yml`
4. `kubectl get ingress`
5. Add the following line to the bottom of the /etc/hosts file:
   `172.17.0.15 hello-world.info`
6. `curl -v http://hello-world.info`
7. `kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0`
8. Edit the existing 01-ingress.yaml and add the following lines:
   ```yaml
      - path: /v2
        pathType: Prefix
        backend:
          service:
            name: web2
            port:
              number: 8080
      ```
9.  `kubectl apply -f example-ingress.yaml`
10. `curl -v http://hello-world.info`
11. `curl hello-world.info/v2`

## Volumes

1. `kubectl get pv -A`
2. `kubectl get pvc -A`
3. `kubectl get storageclass`

## Shared Volume (emptyDir)

1. `kubectl apply -f 02-xchange-pod.yml`
2. `kubectl describe pod sharevol`
3. `kubectl exec -it sharevol -c c1 -- bash`
4. `mount | grep xchange`
5. `echo 'some data' > /tmp/xchange/data`
6. `exit`
7. `kubectl exec -it sharevol -c c2 -- bash`
8. `mount | grep /tmp/data`
9. `cat /tmp/data/data/`
10. `cat /tmp/data/data`
11. `exit`
12. `kubectl delete pod/sharevol`

## PVC

1. `cd 02_pvc`
2. `kubectl apply -f pv.yaml`
3. `kubectl apply -f pvc.yaml`
4. `kubectl get pvc`
5. `kubectl apply -f deploy.yaml`

Now we want to test if data in the volume actually persists. For this we find the pod managed by above deployment, exec into its main container and create a file called data in the /tmp/persistent directory (where we decided to mount the PV)
6. `kubectl exec -it pv-deploy-XXXXX -- bash`
7. `touch /tmp/persistent/data`
8. `ls /tmp/persistent/`
9. `exit`

It’s time to destroy the pod and let the deployment launch a new pod. The expectation is that the PV is available again in the new pod and the data in /tmp/persistent is still present. Let’s check that:
10. `kubectl delete po pv-deploy-XXXXX`
11. `kubectl exec -it pv-deploy-XXXXX -- bash`
12. `ls /tmp/persistent/`
13. `exit`
14. `kubectl delete pvc myclaim`
