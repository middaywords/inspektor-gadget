---
title: Installation
weight: 10
description: >
  How to install.
---

Inspektor Gadget is composed by a `kubectl` plugin executed in the user's
system and a DaemonSet deployed in the cluster.

## Installing kubectl-gadget

Choose one way to install the Inspektor Gadget `kubectl` plugin.

### Using krew

[krew](https://sigs.k8s.io/krew) is the recommended way to install
`kubectl-gadget`. You can follow the
[krew's quickstart](https://krew.sigs.k8s.io/docs/user-guide/quickstart/)
to install it and then install `kubectl-gadget` by executing the following
commands.

```
kubectl krew install gadget
kubectl gadget --help
```

### Install a specific release

Download the asset for a given release and platform from the
[releases page](https://github.com/kinvolk/inspektor-gadget/releases/),
uncompress and move the `kubectl-gadget` executable to your `PATH`.

```
$ curl -sL https://github.com/kinvolk/inspektor-gadget/releases/download/v0.2.0/inspektor-gadget-linux-amd64.tar.gz | sudo tar -C /usr/local/bin -xzf - kubectl-gadget
$ kubectl gadget version
```

### Compile from the sources

```
$ git clone https://github.com/kinvolk/inspektor-gadget.git
$ cd inspektor-gadget
$ make kubectl-gadget-linux-amd64
$ sudo cp kubectl-gadget-linux-amd64 /usr/local/bin/kubectl-gadget
$ kubectl gadget version
```

## Installing in the cluster

### Quick installation

```
$ kubectl gadget deploy | kubectl apply -f -
```

This will deploy the gadget DaemonSet along with its RBAC rules.

### Choosing the gadget image

If you wish to install an alternative gadget image, you could use the following commands:

```
$ kubectl gadget deploy --image=docker.io/myfork/gadget:tag | kubectl apply -f -
```

### Hook Mode

Inspektor Gadget needs to detect when containers are started and stopped.
The different supported modes can be set by using the `hook-mode` option:

- `auto`(default): Inspektor Gadget will try to find the best option based on the system it is running on.
- `crio`: Use the [CRIO hooks](https://github.com/containers/podman/blob/v3.0.0-rc3/pkg/hooks/docs/oci-hooks.5.md) support. Inspektor Gadget installs the required hooks in `/usr/share/containers/oci/hooks.d`, be sure that path is part of the `hooks_dir` option on [crio.conf](https://github.com/cri-o/cri-o/blob/v1.20.0/docs/crio.conf.5.md#crioruntime-table). If `hooks_dir` is not declared at all that path is considered by default.
- `podinformer`: Use a Kubernetes controller to get information about new pods. This option is racy and the first events produced by a container could be lost. This mode is selected when `auto` is used and the above modes are not available.
- `ldpreload`: Adds an entry in `/etc/ld.so.preload` to call a custom shared library that looks for `runc` calls and dynamically adds the needed OCI hooks to the cointainer `config.json` specification. This feature only works when runc is used. Since this feature is highly experimental, it'll not be considered when `auto` is used.
- `nri`: Use the [Node Resource Interface](https://github.com/containerd/nri). It requires containerd v1.5 and it's not considered when `auto` is used.