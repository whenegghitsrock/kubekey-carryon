<div align=center><img src="docs/img/kubekey-logo.svg?raw=true"></div>

[![CI](https://github.com/kubesphere/kubekey/workflows/CI/badge.svg?branch=master&event=push)](https://github.com/kubesphere/kubekey/actions?query=event%3Apush+branch%3Amaster+workflow%3ACI+)

> English | [中文](README_zh-CN.md) | [日本語](README_ja-JP.md)

### 👋 Welcome to KubeKey!

KubeKey is an open-source lightweight tool for deploying Kubernetes clusters. It provides a flexible, rapid, and convenient way to install Kubernetes/K3s only, both Kubernetes/K3s and KubeSphere, and related cloud-native add-ons. It is also an efficient tool to scale and upgrade your cluster.

In addition, KubeKey also supports customized Air-Gap package, which is convenient for users to quickly deploy clusters in offline environments.

> KubeKey has passed [CNCF kubernetes conformance verification](https://www.cncf.io/certification/software-conformance/).

Use KubeKey in the following three scenarios.

* Install Kubernetes/K3s only
* Install Kubernetes/K3s and KubeSphere together in one command
* Install Kubernetes/K3s first, then deploy KubeSphere on it using [ks-installer](https://github.com/kubesphere/ks-installer)

> **Important:** If you have existing Kubernetes clusters, please refer to [ks-installer (Install KubeSphere on existing Kubernetes cluster)](https://github.com/kubesphere/ks-installer).

## Supported Environment

### Linux Distributions

* **Ubuntu**  *16.04, 18.04, 20.04, 22.04*
* **Debian**  *Bookworm, Bullseye, Buster, Stretch*
* **CentOS/RHEL**  *7*
* **AlmaLinux**  *9.0*
* **SUSE Linux Enterprise Server** *15*

> Recommended Linux Kernel Version: `4.15 or later` 
> You can run the `uname -srm` command to check the Linux Kernel Version.

### <span id = "KubernetesVersions">Kubernetes Versions</span>

* **v1.19**: &ensp; *v1.19.15*
* **v1.20**: &ensp; *v1.20.10*
* **v1.21**: &ensp; *v1.21.14*
* **v1.22**: &ensp; *v1.22.15*
* **v1.23**: &ensp; *v1.23.10*   (default)
* **v1.24**: &ensp; *v1.24.7*
* **v1.25**: &ensp; *v1.25.3*

> Looking for more supported versions: \
> [Kubernetes Versions](./docs/kubernetes-versions.md) \
> [K3s Versions](./docs/k3s-versions.md)

### Container Manager

* **Docker** / **containerd** / **CRI-O** / **iSula**

> `Kata Containers` can be set to automatically install and configure runtime class for it when the container manager is containerd or CRI-O.

### Network Plugins

* **Calico** / **Flannel** / **Cilium** / **Kube-OVN** / **Multus-CNI**

> Kubekey also supports users to set the network plugin to `none` if there is a requirement for custom network plugin.

## Requirements and Recommendations

* Minimum resource requirements (For Minimal Installation of KubeSphere only)：
  * 2 vCPUs
  * 4 GB RAM
  * 20 GB Storage

> /var/lib/docker is mainly used to store the container data, and will gradually increase in size during use and operation. In the case of a production environment, it is recommended that /var/lib/docker mounts a drive separately.

* OS requirements:
  * `SSH` can access to all nodes.
  * Time synchronization for all nodes.
  * `sudo`/`curl`/`openssl` should be used in all nodes.
  * `docker` can be installed by yourself or by KubeKey.
  * `Red Hat` includes `SELinux` in its `Linux release`. It is recommended to close SELinux or [switch the mode of SELinux](./docs/turn-off-SELinux.md) to `Permissive`

> * It's recommended that Your OS is clean (without any other software installed), otherwise there may be conflicts.
> * A container image mirror (accelerator) is recommended to be prepared if you have trouble downloading images from dockerhub.io. [Configure registry-mirrors for the Docker daemon](https://docs.docker.com/registry/recipes/mirror/#configure-the-docker-daemon).
> * KubeKey will install [OpenEBS](https://openebs.io/) to provision LocalPV for development and testing environment by default, this is convenient for new users. For production, please use NFS / Ceph / GlusterFS  or commercial products as persistent storage, and install the [relevant client](docs/storage-client.md) in all nodes.
> * If you encounter `Permission denied` when copying, it is recommended to check [SELinux and turn off it](./docs/turn-off-SELinux.md) first

* Dependency requirements:

KubeKey can install Kubernetes and KubeSphere together. Some dependencies need to be installed before installing kubernetes after version 1.18. You can refer to the list below to check and install the relevant dependencies on your node in advance.

|               | Kubernetes Version ≥ 1.18 |
| ------------- | -------------------------- |
| `socat`     | Required                   |
| `conntrack` | Required                   |
| `ebtables`  | Optional but recommended   |
| `ipset`     | Optional but recommended   |
| `ipvsadm`   | Optional but recommended   |

* Networking and DNS requirements:
  * Make sure the DNS address in `/etc/resolv.conf` is available. Otherwise, it may cause some issues of DNS in cluster.
  * If your network configuration uses Firewall or Security Group，you must ensure infrastructure components can communicate with each other through specific ports. It's recommended that you turn off the firewall or follow the link configuriation: [NetworkAccess](docs/network-access.md).

## Usage

### Get the KubeKey Executable File

* The fastest way to get KubeKey is to use the script:

  ```
  curl -sfL https://get-kk.kubesphere.io | sh -
  ```
* Binary downloads of the KubeKey also can be found on the [Releases page](https://github.com/kubesphere/kubekey/releases).
  Unpack the binary and you are good to go!
* Build Binary from Source Code

  ```shell
  git clone https://github.com/kubesphere/kubekey.git
  cd kubekey
  make kk
  ```

### Create a Cluster

#### Quick Start

Quick Start is for `all-in-one` installation which is a good start to get familiar with Kubernetes and KubeSphere.

> Note: Since Kubernetes temporarily does not support uppercase NodeName, contains uppercase letters in the hostname will lead to subsequent installation error

##### Command

> If you have problem to access `https://storage.googleapis.com`, execute first `export KKZONE=cn`.

```shell
./kk create cluster [--with-kubernetes version] [--with-kubesphere version]
```

##### Examples

* Create a pure Kubernetes cluster with default version (Kubernetes v1.23.10).

  ```shell
  ./kk create cluster
  ```
* Create a Kubernetes cluster with a specified version.

  ```shell
  ./kk create cluster --with-kubernetes v1.24.1 --container-manager containerd
  ```
* Create a Kubernetes cluster with KubeSphere installed.

  ```shell
  ./kk create cluster --with-kubesphere v3.2.1
  ```

#### Advanced

You have more control to customize parameters or create a multi-node cluster using the advanced installation. Specifically, create a cluster by specifying a configuration file.

> If you have problem to access `https://storage.googleapis.com`, execute first `export KKZONE=cn`.

1. First, create an example configuration file

   ```shell
   ./kk create config [--with-kubernetes version] [--with-kubesphere version] [(-f | --filename) path]
   ```

   **examples:**

   * create an example config file with default configurations. You also can specify the file that could be a different filename, or in different folder.

   ```shell
   ./kk create config [-f ~/myfolder/abc.yaml]
   ```

   * with KubeSphere

   ```shell
   ./kk create config --with-kubesphere v3.2.1
   ```
2. Modify the file config-sample.yaml according to your environment

> Note:  Since Kubernetes temporarily does not support uppercase NodeName, contains uppercase letters in workerNode`s name will lead to subsequent installation error
>
> A persistent storage is required in the cluster, when kubesphere will be installed. The local volume is used default. If you want to use other persistent storage, please refer to [addons](./docs/addons.md).

3. Create a cluster using the configuration file

   ```shell
   ./kk create cluster -f config-sample.yaml
   ```

### Enable Multi-cluster Management

By default, KubeKey will only install a **solo** cluster without Kubernetes federation. If you want to set up a multi-cluster control plane to centrally manage multiple clusters using KubeSphere, you need to set the `ClusterRole` in [config-example.yaml](docs/config-example.md). For multi-cluster user guide, please refer to [How to Enable the Multi-cluster Feature](https://github.com/kubesphere/community/tree/master/sig-multicluster/how-to-setup-multicluster-on-kubesphere).

### Enable Pluggable Components

KubeSphere has decoupled some core feature components since v2.1.0. These components are designed to be pluggable which means you can enable them either before or after installation. By default, KubeSphere will be started with a minimal installation if you do not enable them.

You can enable any of them according to your demands. It is highly recommended that you install these pluggable components to discover the full-stack features and capabilities provided by KubeSphere. Please ensure your machines have sufficient CPU and memory before enabling them. See [Enable Pluggable Components](https://github.com/kubesphere/ks-installer#enable-pluggable-components) for the details.

### Add Nodes

Add new node's information to the cluster config file, then apply the changes.

```shell
./kk add nodes -f config-sample.yaml
```

### Delete Nodes

You can delete the node by the following command，the nodeName that needs to be removed.

```shell
./kk delete node <nodeName> -f config-sample.yaml
```

### Delete Cluster

You can delete the cluster by the following command:

* If you started with the quick start (all-in-one):

```shell
./kk delete cluster
```

* If you started with the advanced (created with a configuration file):

```shell
./kk delete cluster [-f config-sample.yaml]
```

### Upgrade Cluster

#### Allinone

Upgrading cluster with a specified version.

```shell
./kk upgrade [--with-kubernetes version] [--with-kubesphere version] 
```

* Support upgrading Kubernetes only.
* Support upgrading KubeSphere only.
* Support upgrading Kubernetes and KubeSphere.

#### Multi-nodes

Upgrading cluster with a specified configuration file.

```shell
./kk upgrade [--with-kubernetes version] [--with-kubesphere version] [(-f | --filename) path]
```

* If `--with-kubernetes` or `--with-kubesphere` is specified, the configuration file will be also updated.
* Use `-f` to specify the configuration file which was generated for cluster creation.

> Note: Upgrading multi-nodes cluster need a specified configuration file. If the cluster was installed without kubekey or the configuration file for installation was not found, the configuration file needs to be created by yourself or following command.

Getting cluster info and generating kubekey's configuration file (optional).

```shell
./kk create config [--from-cluster] [(-f | --filename) path] [--kubeconfig path]
```

* `--from-cluster` means fetching cluster's information from an existing cluster.
* `-f` refers to the path where the configuration file is generated.
* `--kubeconfig` refers to the path where the kubeconfig.
* After generating the configuration file, some parameters need to be filled in, such as the ssh information of the nodes.

## Documents

* [Features List](docs/features.md)
* [Commands](docs/commands/kk.md)
* [Configuration example](docs/config-example.md)
* [Air-Gapped Installation](docs/manifest_and_artifact.md)
* [Highly Available clusters](docs/ha-mode.md)
* [Addons](docs/addons.md)
* [Network access](docs/network-access.md)
* [Storage clients](docs/storage-client.md)
* [kubectl auto-completion](docs/kubectl-autocompletion.md)
* [kubekey auto-completion](docs/kubekey-autocompletion.md)
* [Roadmap](docs/roadmap.md)
* [Check-Renew-Certificate](docs/check-renew-certificate.md)
* [Developer-Guide](docs/developer-guide.md)

## 💖 Support this Project  

If you find this project useful, consider buying me a coffee ☕️.  

### 💰 Crypto Donation  

- **BTC Address:** bc1qn3d92gg6v8p78ej680gr7xf7q6d73alu9fga4t
  ![BTC QR Code](./donation/btc.png)  

- **ETH Address:** 0x6433B4d3963361037734fd34dEA7D3683F36D291
  ![ETH QR Code](./donation/eth.png)  

- **SOLANA Address:** 7XY3icFFrm9rcHFDBjMQhnuZ2d7jF4xD4TA8wKU9SZW
  ![SOLANA QR Code](./donation/sol.png)  

