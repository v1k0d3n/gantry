[![Build Status](http://jenkins.jinkit.com:8080/buildStatus/icon?job=gantry)](http://jenkins.jinkit.com:8080/job/gantry/)<br>
[![Docker Repository on Quay](https://quay.io/repository/v1k0d3n/gantry/status "Docker Repository on Quay")](https://quay.io/repository/v1k0d3n/gantry)

# Gantry: a containerized kubeadm project
A container that bootstraps Kubernetes using Kubeadm (containerized).

**WARNING: THIS REPO IS A WIP** <br>
This is just a working start, but not how the project will be used as an end state. The plan is to put all logic in the `gantry` initially, to determine distro (for required mounts and placement), state (bootstrap, clean, etc), and potentially considerations for some common plugins or options (Helm, IPVS, etc).

## Basic Usage:
If you want to try this as its in early stages, you can use the container like this:

1. Builds the container like below. You can leveage `--build-args` to customize your image. 
```shell
git clone https://github.com/v1k0d3n/gantry.git
cd gantry 
export KUBE_VERSION=v1.10.0
sudo docker build --build-arg VERSION_KUBEADM=${KUBE_VERSION} --build-arg VERSION_KUBECTL=${KUBE_VERSION} --build-arg VERSION_KUBELET=${KUBE_VERSION} -t gantry:${KUBE_VERSION} .
```

2. Then start the container with the following parameters (this is likely to change as the project is being tested): <br>
**NOTE:** for `$(pwd)` in the line `-v $(pwd):/kubeadm/etc/kubeadm`, this should be the location of your kubeadm `MasterConfiguration` yaml manifest.
```shell
sudo rm -rf /opt/kubeadm
sudo docker run -it \
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
   gantry:${KUBE_VERSION} gantry -d ubuntu -i --config /kubeadm/etc/kubeadm/config.yaml
```
Container images of Gantry are available on both [DockerHub](https://hub.docker.com/r/v1k0d3n/gantry/tags/) and [Quay](https://quay.io/repository/v1k0d3n/gantry?tab=tags).

**NOTE:** The intention of Gantry is to declaratively bootstrap a Kubernetes cluster using a custom Kubeadm `MasterConfiguration` file. The Gantry image includes a [sample config](https://github.com/v1k0d3n/gantry/blob/master/etc/kubeadm/config.yaml), but we recommend reading the documentation for bootstrapping [kubeadm with configuration file](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file).

3. You can destroy a previously bootstrapped cluster by using `gantry -r`. Please refer to the `--help` menu for any questions on how to use the Gantry image.

4. After bootstrapping a cluster with Gantry/Kubeadm, you will still need to configure `kubectl` and apply an SDN manifest:
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

## Alternative Methods:
If you don't want to use Gantry to bootstrap your cluster, you can still use the Gantry image to distribute Kubernetes binaries ([kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/), [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/), and [kubelet](https://kubernetes.io/docs/reference/generated/kubelet/)). All of the binaries are being downloaded directly from [Kubernetes releases](https://storage.googleapis.com/kubernetes-release/) and they are located in `/kubeadm/bin/`. A Gantry image will be created for each Kubernetes release. Simply copy them directly to your host, and use them for your specific setup.

```shell
ubuntu@gantry-test:~$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
97a68d1dd91b        gantry:v1.10.0      "gantry -h"         24 minutes ago      Exited (0) 2  minutes ago                       reverent_mccarthy
ubuntu@gantry-test:~$ docker cp 97a68d1dd91b:/kubeadm/bin/kubeadm ~
ubuntu@gantry-test:~$ ls -asl ~/kubeadm
152804 -rwxr-xr-x 1 ubuntu ubuntu 156467952 Mar 31 04:28 /home/ubuntu/kubeadm
ubuntu@gantry-test:~$
``` 

## Future State:
I would really like to get to a future-state that [Jessie Frazelle](https://github.com/jessfraz/) is promoting on her [blog](https://blog.jessfraz.com/) which [builds images securely](https://blog.jessfraz.com/post/building-container-images-securely-on-kubernetes/). We can try to improve the need to run full `--privileged` flags in the meantime. This isn't desired, but is easiest for now.

## Contributing, Comments, Questions
Comments, suggestions and PR's are welcome!
