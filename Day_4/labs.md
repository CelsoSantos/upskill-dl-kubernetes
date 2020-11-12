# Upskill Labs - Day 4

## Clone

git clone https://github.com/CelsoSantos/upskill-dl-kubernetes.git

## DaemonSet

1. `kubectl apply -f daemonset/01-daemonset.yaml`
2. `kubectl get nodes -o wide`
3. `kubectl apply -f daemonset/02-daemonset.yaml`

## CRD

1. `cd crd/`
2. `kubectl get crd -A`
3. `kubectl apply -f crd.yaml`
4. `kubectl apply -f appconfig.yaml`
5. `kubectl get ac -A`

## Operator

Source: https://strimzi.io/quickstarts/

1. `kubectl create namespace kafka`
2. `kubectl apply -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka`
3. `kubectl apply -f https://strimzi.io/examples/latest/kafka/kafka-persistent-single.yaml -n kafka`
4. `kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n kafka`

Send and receive Messages:

1. `kubectl -n kafka run kafka-producer -ti --image=strimzi/kafka:0.20.0-kafka-2.6.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic my-topic`
2. `kubectl -n kafka run kafka-consumer -ti --image=strimzi/kafka:0.20.0-kafka-2.6.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning`