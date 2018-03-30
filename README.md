[![Build Status](http://jenkins.jinkit.com:8080/buildStatus/icon?job=gantry)](http://jenkins.jinkit.com:8080/job/gantry/)<br>
[![Docker Repository on Quay](https://quay.io/repository/v1k0d3n/kubeadm-contained/status "Docker Repository on Quay")](https://quay.io/repository/v1k0d3n/kubeadm-contained) [![Docker Hub Repository](https://dockerbuildbadges.quelltext.eu/status.svg?organization=v1k0d3n&repository=kubeadm-contained)](https://hub.docker.com/r/v1k0d3n/kubeadm-contained/tags/)
# Gantry: a containerized kubeadm project
A container that bootstraps Kubernetes using Kubeadm (containerized).

**WARNING: THIS REPO IS A WIP**<br>
This is just a working start, but not how the project will be used as an end state. The plan is to put all logic in the `gantry` initially, to determine distro (for required mounts and placement), state (bootstrap, clean, etc), and potentially considerations for some common plugins or options (Helm, IPVS, etc).

## Basic Usage:
If you want to try this as its in early stages, you can use the container like this:

1. Builds the container like so:
```shell
git clone https://github.com/v1k0d3n/gantry.git
cd gantry 
sudo docker build -t kubeadm-contained .
```

2. Then start the container with the following parameters (this is likely to change as the project is being tested):
<br>***NOTE:*** *`-v $(pwd):/kubeadm/etc/kubeadm` should be the location of your kubeadm `MasterConfiguration` yaml.*
```shell
sudo rm -rf /opt/kubeadm
sudo docker run -d \
   --privileged \
   --net=host \
   -v /etc/cni:/etc/cni \
   -v /var/lib/etcd:/var/lib/etcd \
   -v /etc/kubernetes:/etc/kubernetes \
   -v /usr/libexec/kubernetes:/usr/libexec/kubernetes \
   -v /var/lib/kubelet:/var/lib/kubelet \
   -v /usr/bin/systemctl:/usr/bin/systemctl \
   -v /etc/systemd/system:/etc/systemd/system \
   -v /var/run/docker.sock:/var/run/docker.sock \
   -v /lib/modules:/lib/modules \
   -v /var/run:/var/run \
   -v /usr/bin:/usr/bin \
   -v /boot:/boot \
   -v /opt:/opt \
   -v $(pwd):/kubeadm/etc/kubeadm \
   kubeadm-contained gantry -h
```
There are also containers available from [DockerHub](https://hub.docker.com/r/v1k0d3n/kubeadm-contained/tags/) and [Quay](https://quay.io/repository/v1k0d3n/kubeadm-contained?tab=tags).


3. You can bring up a cluster with the following syntax (which is still, very much a WIP):
```shell
gantry -d ubuntu -i --config /kubeadm/etc/kubeadm/YOUR_KUBEADM_MANIFEST.yaml 
```

**NOTE:** If you want to deploy with a custom `kubeadm` `MasterConfiguration` file, mount `/kubeadm/etc/kubeadm/` to the directory where your config is located. There's a [sample config](https://github.com/v1k0d3n/gantry/blob/master/etc/kubeadm/config.yaml) included, but please refer to section [kubeadm init with a configuration file](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file) section of the Kubernetes documentation.

4. As per the `--help` menu, you can destroy a running cluster by using: `gantry -r`

5. You will still need to configure `kubectl` and apply an SDN manifest (configure your cluster accordingly):
```shell
# Configure kubectl:
mkdir -p $HOME/.kube
yes | sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# If testing:
kubectl taint nodes --all node-role.kubernetes.io/master-

# Apply SDN (Calico example):
kubectl apply -f https://gist.githubusercontent.com/v1k0d3n/aa318f52399f5ebdd6043dd615ae07b4/raw/ed583598170d67bc8c6c91dc523ce100482958eb/networking-calico.yaml
```

## Preparation:
Docker is the only real requirement to run this kubeadm-containerized image. If you have a new or default installation (currently Ubuntu Xenial: 16.04), you can use the preparation script to install Docker.

On the Kubernetes node (where this will be deployed), run the following from the main `gantry` directory:
```shell
./bin/distro/ubuntu/start
```

## Contributing, Comments, Questions
Comments, suggestions and PR's are welcome!
