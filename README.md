# Open Source MANO kubernetes deployment

K8s deployment for Open Source MANO(OSM).

## Current state of development

* Currently only deployed in All in one using minikube,
some issues exists with kafka topics which makes OSM unusable for now.
* Some bugs are present in the docker images, other things are hardcoded
which makes the deployment kind of weird and not much configurable.
* LCM and POl services are restarting in CrashLoopBackOff state.

Give some time for testing and contribute in you feel it ;)

## Architecture design discussion and items to be work

Goal of this project is to deploy an OSM in HA multi-cluster within k8s.

###Main requirements

* Database replication (mongodb and mariadb)
* MQ/data HA (zookeeper and kafka)
* APIs active/active HA

To achieve this goal is probably the best option to use already existing
k8s deployments for kafka, mongodb,zookeeper and prometheus.

Let do the experts in the area do their work, and we do our job to deploy OSM.

### Work items

* Fix docker images to be more configurable and less attached to Swarm deployment.
  Others deployer will benefit from this
* Healthcheck
* Stateful deployment for data services
* Service dependencies, wait for previous to be available with initContainers
parameter
* Some kind of script to configure a few parameters like passwords
* Ability to configure different service types, as LB, NodePort, etc
* Helm charts (Maybe?)

## Quickstart k8s installation
### Install minikube - KVM driver

```bash
sudo usermod -a -G libvirt $(whoami)
curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-kvm2 \
    && sudo install docker-machine-driver-kvm2 /usr/local/bin/

sudo wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 -O  /usr/local/bin/minikube
sudo chmod +x /usr/local/bin/minikube

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
curl -L https://github.com/kubernetes/kompose/releases/download/v1.16.0/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose

minikube start --vm-driver kvm2 --memory 8096
```

### Verify minikube
```bash
kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
kubectl expose deployment hello-minikube --type=NodePort
curl $(minikube service hello-minikube --url)
```

## Open Source MANO deployment
### Required osm images

* wurstmeister/zookeeper:latest
* wurstmeister/kafka
* mongo
* prom/prometheus
* mariadb:10
* opensourcemano/keystone
* opensourcemano/lcm
* opensourcemano/nbi
* mysql:5
* opensourcemano/ro
* opensourcemano/mon
* opensourcemano/pol
* opensourcemano/light-ui


### Deploy OSM
Deploy one by one, wait for the previous service to be available.
```bash
kubectl apply -f osm-zookeeper.yaml
kubectl apply -f osm-kafka.yaml
kubectl apply -f osm-mongo.yaml
kubectl apply -f osm-prometheus.yaml
kubectl apply -f osm-keystone-db.yaml
kubectl apply -f osm-keystone.yaml
kubectl apply -f osm-nbi.yaml
kubectl apply -f osm-ro-db.yaml
kubectl apply -f osm-ro.yaml
kubectl apply -f osm-lcm.yaml
kubectl apply -f osm-mon.yaml
kubectl apply -f osm-pol.yaml
kubectl apply -f osm-light-ui.yaml
```

### Web UI user password
```bash
admin:admin
```

### Bonus point
##### Create tunnel to service's IP to allow external traffic (when not using LB)
```bash
ssh -i ~/.minikube/machines/minikube/id_rsa docker@$(minikube ip) -L <local_port>:<service_ip>:<service_port>

curl http://$(minikube ip):<local_port>
```

## License

Apache version 2

## Contact information

Feel free to reach me at ``dabarren@gmail.com`` or as ``egonzalez`` at OSM slack channel
or at ```#openstack-kolla``` IRC channel at freenode.org
