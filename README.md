# Deploying Spark on Kubernetes

## Want to learn how to build this?

Check out the [post](https://testdriven.io/deploying-spark-on-kubernetes).

## Want to use this project?

### Minikube Setup

Install and run [Minikube](https://kubernetes.io/docs/setup/minikube/):

1. Install a  [Hypervisor](https://kubernetes.io/docs/tasks/tools/install-minikube/#install-a-hypervisor) (like [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or [HyperKit](https://github.com/moby/hyperkit)) to manage virtual machines
1. Install and Set Up [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to deploy and manage apps on Kubernetes
1. Install [Minikube](https://github.com/kubernetes/minikube/releases)

Start the cluster:

```sh
$ minikube start --memory 8192 --cpus 4
$ minikube dashboard
```

Build the Docker image:

```sh
$ eval $(minikube docker-env)
$ docker build -t spark-hadoop:2.2.1 -f ./docker/Dockerfile ./docker
```

Create the deployments and services:

```sh
$ kubectl create -f ./kubernetes/spark-master-deployment.yaml
$ kubectl create -f ./kubernetes/spark-master-service.yaml
$ kubectl create -f ./kubernetes/spark-worker-deployment.yaml
$ minikube addons enable ingress
$ kubectl apply -f ./kubernetes/minikube-ingress.yaml
```

Add an entry to /etc/hosts:

```sh
$ echo "$(minikube ip) spark-kubernetes" | sudo tee -a /etc/hosts
```

In **Mac with minikube** installed, you must execute this command and create a tunnel like this

We must patch the original Dockerfile

```
#ADD common.sh spark-master spark-worker /
COPY common.sh spark-master spark-worker /
RUN chmod +x /common.sh /spark-master /spark-worker
```

we must add this entrance in the hosts file to create a static dns resolution
from 127.0.0.1 to spark-kubernetes dns controlled by the ingress controller inside kubernetes

```sh
$ echo "127.0.0.1 spark-kubernetes" | sudo tee -a /etc/hosts

$ minikube tunnel
```

Now open the uri from your browser

```sh
http://spark-kubernetes
```

![Spark UI](./images/spark_ui.png "spark UI")

Execute the terminal pyspark inside the master node, to recover bindAddress execute first:
```sh
$ kubectl get pods -o wide

NAME                            READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
spark-master-6bc899886b-m6r82   1/1     Running   0          9m57s   10.244.0.15   minikube   <none>           <none>
spark-worker-6f954cb794-g6g9n   1/1     Running   0          8m50s   10.244.0.16   minikube   <none>           <none>
spark-worker-6f954cb794-km2sp   1/1     Running   0          8m50s   10.244.0.17   minikube   <none>           <none>

$ kubectl exec spark-master-6bc899886b-m6r82 -it -- \pyspark --conf spark.driver.bindAddress=10.244.0.15 --conf spark.driver.host=10.244.0.15
```

Test it out in the browser at [http://spark-kubernetes/](http://spark-kubernetes/).
