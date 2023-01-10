
# How to setup a minikube and docker on ubuntu environment before Kubernetes v1.25.0 Training



## Prerequisits : 
Every participant must have a Virtual Machine with the following requirements :
- Ubuntu 22.04 LTS (desktop)
- 4 vCPU / 16 GB RAM
- Firefox 106.0.2 (64-bit) must be installed 
- User must be sudoer
- User must have RDP access to Ubuntu UI
- Egress traffic must be authorized to 0.0.0.0/0 (Github, DockerHub, RMP, ... etc)
#
### 1 - Install Docker 20.10.21 and containerd
**Steps :**

Please follow the official documentation : https://docs.docker.com/engine/install/ubuntu/#set-up-the-repository

**Verification:**
```sh
root@minikube: docker version
Client: Docker Engine - Community
 Version:           20.10.21
 API version:       1.41
 Go version:        go1.18.7
 Git commit:        baeda1f
 Built:             Tue Oct 25 18:01:58 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.21
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.18.7
  Git commit:       3056208
  Built:            Tue Oct 25 17:59:49 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.9
  GitCommit:        1c90a442489720eec95342e1789ee8a5e1b9536f
 runc:
  Version:          1.1.4
  GitCommit:        v1.1.4-0-g5fd4c4d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

```

#
### 2- Install cri-docker v0.2.6
**Steps :**
- Optiom 1 _(recommended)_: Clone source code and build binary

    Please follow the official documentation : https://github.com/Mirantis/cri-dockerd#build-and-install

    _Steps:_

    a- clone source code : 
    ```sh
    git clone https://github.com/Mirantis/cri-dockerd.git
    ```
    b- build and install the binary

    ```sh
    # Run these commands as root
    ###Install GO###
    wget https://storage.googleapis.com/golang/getgo/installer_linux
    chmod +x ./installer_linux
    ./installer_linux
    source ~/.bash_profile

    cd cri-dockerd
    mkdir bin
    go build -o bin/cri-dockerd
    mkdir -p /usr/local/bin
    install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
    cp -a packaging/systemd/* /etc/systemd/system
    sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
    systemctl daemon-reload
    systemctl start cri-docker.service
    systemctl enable cri-docker.service
    systemctl enable --now cri-docker.socket
    ```

- Option 2 : Install debian packages from this release : https://github.com/Mirantis/cri-dockerd/releases/tag/v0.2.6

**Verification:**

```sh
root@minikube: service cri-docker status
    ● cri-docker.service - CRI Interface for Docker Application Container Engine
        Loaded: loaded (/etc/systemd/system/cri-docker.service; enabled; vendor preset: enabled)
        Active: active (running) since Thu 2022-10-27 12:05:29 UTC; 18min ago
    TriggeredBy: ● cri-docker.socket
        Docs: https://docs.mirantis.com
    Main PID: 1177 (cri-dockerd)
        Tasks: 9
        Memory: 40.0M
            CPU: 382ms
        CGroup: /system.slice/cri-docker.service
                └─1177 /usr/local/bin/cri-dockerd --container-runtime-endpoint fd://
    Oct 27 12:05:29 minikube cri-dockerd[1177]: time="2022-10-27T12:05:29Z" level=info msg="Start docker client with request timeout 0s"
    Oct 27 12:05:29 minikube cri-dockerd[1177]: time="2022-10-27T12:05:29Z" level=info msg="Hairpin mode is set to none"
    Oct 27 12:05:29 minikube cri-dockerd[1177]: time="2022-10-27T12:05:29Z" level=info msg="Loaded network plugin cni"
    Oct 27 12:05:29 minikube cri-dockerd[1177]: time="2022-10-27T12:05:29Z" level=info msg="Docker cri networking managed by network plugin cni"
    Oct 27 12:05:29 minikube cri-dockerd[1177]: time="2022-10-27T12:05:29Z" level=info msg="Docker Info: &{ID:7BCA:BFH3:BRNK:IVZN:YDHJ:UK3U:UV5V:DBKO:L75C:RPHN:54Z>
    Oct 27 12:05:29 minikube cri-dockerd[1177]: time="2022-10-27T12:05:29Z" level=info msg="Setting cgroupDriver systemd"
    Oct 27 12:05:29 minikube cri-dockerd[1177]: time="2022-10-27T12:05:29Z" level=info msg="Docker cri received runtime config &RuntimeConfig{NetworkConfig:&Networ>
    Oct 27 12:05:29 minikube cri-dockerd[1177]: time="2022-10-27T12:05:29Z" level=info msg="Starting the GRPC backend for the Docker CRI interface."
    Oct 27 12:05:29 minikube cri-dockerd[1177]: time="2022-10-27T12:05:29Z" level=info msg="Start cri-dockerd grpc backend"
    Oct 27 12:05:29 minikube systemd[1]: Started CRI Interface for Docker Application Container Engine.
 ```

#
### 3- Install kubectl v1.25.0
**Steps :**

Please follow the official documentation : https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux

**Verification:**
```sh
root@minikube: kubectl version --short --client
Flag --short has been deprecated, and will be removed in the future. The --short output will become the default.
Client Version: v1.25.0
Kustomize Version: v4.5.7
```

#
### 4- Install minikube v1.27.1
**Steps :**

Please follow the official documentation : https://kubernetes.io/fr/docs/tasks/tools/install-minikube/

**Verification:**
```sh
root@minikube: minikube version
minikube version: v1.27.1
commit: fe869b5d4da11ba318eb84a3ac00f336411de7ba
```
#
### 5- Start minikube
**Steps :**
```sh
root@minikube: minikube start --driver=docker --container-runtime=containerd --cpus 4 --memory 8192 --force
😄  minikube v1.27.1 on Ubuntu 22.04 (amd64)
❗  minikube skips various validations when --force is supplied; this may lead to unexpected behavior
✨  Using the docker driver based on existing profile
🛑  The "docker" driver should not be used with root privileges. If you wish to continue as root, use --force.
💡  If you are running minikube within a VM, consider using --driver=none:
📘    https://minikube.sigs.k8s.io/docs/reference/drivers/none/
💡  Tip: To remove this root owned cluster, run: sudo minikube delete
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔄  Restarting existing docker container for "minikube" ...
📦  Preparing Kubernetes v1.25.2 on containerd 1.6.8 ...
🔗  Configuring CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by defaul
``` 
_* In case of failure please run the following command :_
```
minikube delete
sudo sysctl fs.protected_regular=0
```
_* PS :  Minikube will be deleted automatically after every VM shutdown_

**Verification:**
```sh
root@minikube: minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configure
```

```sh
root@minikube: docker ps
CONTAINER ID   IMAGE                                 COMMAND                  CREATED        STATUS          PORTS                                                                                                                                  NAMES
93973ae38df8   gcr.io/k8s-minikube/kicbase:v0.0.35   "/usr/local/bin/entr…"   22 hours ago   Up 17 minutes   127.0.0.1:49157->22/tcp, 127.0.0.1:49156->2376/tcp, 127.0.0.1:49155->5000/tcp, 127.0.0.1:49154->8443/tcp, 127.0.0.1:49153->32443/tcp   minikube
```

```sh
root@minikube: kubectl get po -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-565d847f94-cnfcr           1/1     Running   2 (18m ago)   21h
kube-system   etcd-minikube                      1/1     Running   2 (18m ago)   21h
kube-system   kindnet-5k74l                      1/1     Running   2 (18m ago)   21h
kube-system   kube-apiserver-minikube            1/1     Running   2 (18m ago)   21h
kube-system   kube-controller-manager-minikube   1/1     Running   2 (18m ago)   21h
kube-system   kube-proxy-4wxdv                   1/1     Running   2 (18m ago)   21h
kube-system   kube-scheduler-minikube            1/1     Running   2 (18m ago)   21h
kube-system   storage-provisioner                1/1     Running   4 (17m ago)   21h
```

### Authors

* **Oussama BEN CHARRADA** - *Initial work*
* &copy; [SETCLOUDUP]( https://setcloudup.com/)
