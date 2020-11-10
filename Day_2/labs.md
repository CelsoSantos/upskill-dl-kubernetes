# Upskill Labs - Day 2

## Start

1. kubectl get pv -A
2. kubectl get pvc -A
3. kubectl get storageclass

## Shared Volume (emptyDir)

1. kubectl apply -f 01-xchange-pod.yml
2. kubectl describe pod sharevol
3. kubectl exec -it sharevol -c c1 -- bash
4. mount | grep xchange
5. echo 'some data' > /tmp/xchange/data
6. exit
7. kubectl exec -it sharevol -c c2 -- bash
8. mount | grep /tmp/data
9. cat /tmp/data/data/
10. cat /tmp/data/data
11. exit
12. kubectl delete pod/sharevol

## PVC

1. cd 02_pvc
2. kubectl apply -f pv.yaml
3. kubectl apply -f pvc.yaml
4. kubectl get pvc
5. kubectl apply -f deploy.yaml
6. Now we want to test if data in the volume actually persists. For this we find the pod managed by above deployment, exec into its main container and create a file called data in the /tmp/persistent directory (where we decided to mount the PV)
7. kubectl exec -it pv-deploy-XXXXX -- bash
8. touch /tmp/persistent/data
9. ls /tmp/persistent/
10. exit
11. It’s time to destroy the pod and let the deployment launch a new pod. The expectation is that the PV is available again in the new pod and the data in /tmp/persistent is still present. Let’s check that:
12. kubectl delete po pv-deploy-XXXXX
13. kubectl exec -it pv-deploy-XXXXX -- bash
14. ls /tmp/persistent/
15. exit
16. kubectl delete pvc myclaim
