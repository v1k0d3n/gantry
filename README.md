[![Docker Repository on Quay](https://quay.io/repository/v1k0d3n/gantry/status "Docker Repository on Quay")](https://quay.io/repository/v1k0d3n/gantry)
# Gantry: A Kubeadm Container Project
A container that bootstraps Kubernetes using Kubeadm (containerized).

**WARNING: THIS REPO IS A WIP**<br>
This is just a working start, but not how the project will be used as an end state. The plan is to put all logic in the `entrypoint.sh` to determine distro (for required mounts and placement), state (bootstrap, clean, etc), and some considerations for plugins or options (Helm, IPVS, et).

## Basic Usage:
If you want to try this as its in early stages, you can use it like this:

1. Builds the container like so:
```shell
git clone https://github.com/v1k0d3n/gantry.git
cd gantry 
sudo docker build -t kubeadm-contained .
```

2. Start the container with the following parameters (this is likely to change as the project is being tested):
```shell
sudo rm -rf /opt/kubeadm
sudo docker run -it \
   --privileged \
   --net=host \
   --security-opt seccomp:unconfined \
   --cap-add=ALL \
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
   kubeadm-contained gantry -h
```
There are also containers available from DockerHub and Quay under: [v1k0d3n/gantry](https://quay.io/repository/v1k0d3n/gantry?tab=tags)


3. You can bring up a cluster with the following syntax (still a WIP):
```shell
gantry -d centos -i --config /opt/kubeadm/etc/kubeadm/config.yaml
```
And you can destroy a running cluster by using: `gantry -r`

**NOTE:** If you want to deploy with a custom `kubeadm` `MasterConfiguration` file, move your config to `/opt/kubeadm/etc/kubeadm/`.

4. You will still need to configure `kubectl` and apply an SDN manifest (configure your cluster accordingly):
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
