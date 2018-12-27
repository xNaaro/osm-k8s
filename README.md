# Intro

K8s deployment for Open Source MANO(OSM).

#### Current state of development

* Currently only deployed in All in one using minikube,
some issues exists with kafka topics which makes OSM unusable for now.

* Give some time for testing and contribute in you feel it ;)
#test protected branch

# Install minikube - KVM driver

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

# Verify minikube
```bash
kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
kubectl expose deployment hello-minikube --type=NodePort
curl $(minikube service hello-minikube --url)
```

# Required osm images

* docker pull wurstmeister/zookeeper:latest
* docker pull wurstmeister/kafka
* docker pull mongo
* docker pull prom/prometheus
* docker pull mariadb:10
* docker pull opensourcemano/keystone
* docker pull opensourcemano/lcm
* docker pull opensourcemano/nbi
* docker pull mysql:5
* docker pull opensourcemano/ro
* docker pull opensourcemano/mon
* docker pull opensourcemano/pol
* docker pull opensourcemano/light-ui


# Deploy OSM
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

# Bonus point
#### Create tunnel to service's IP to allow external traffic (when not using LB)
```bash
ssh -i ~/.minikube/machines/minikube/id_rsa docker@$(minikube ip) -L <local_port>:<service_ip>:<service_port>

curl http://$(minikube ip):<local_port>
```
Feel free to reach me at ``dabarren@gmail.com`` or as ``egonzalez`` at OSM slack channel
or at ```#openstack-kolla``` IRC channel at freenode.org
