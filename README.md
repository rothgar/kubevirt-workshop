# Talos on KubeVirt workshop

This repo will get you started running a Talos on Talos workshop with KubeVirt.

The main components are:

1. Base Kubernetes cluster (with KubeVirt installed)
1. Talos VM Pool
1. (optional) local web mirror
1. (optional) local registry mirror
1. (optional) KubeVirt manager - web UI

This workshop can also be used for creating Ubuntu VMs used for running Omni locally with an Ubuntu VM pool.

## Base cluster reqirements

The base cluster can be created with Omni or manually with Talos.
You will need to have a bridge adapter (br0) so VMs can get an IP address from the LAN network and users can access VMs directly without proxying the pod network.

There is a patch for nodes in the [base-cluster/br0-patch.yaml](base-cluster/br0-patch.yaml) file.

## Talos VM pool

Nodes will be created with a network interface using the br0 adapter and will get an IP address directly from the LAN.
With the travel router nodes will get IP addresses above 10.10.1.100.
Everything below .100 is available for static IP addresses and VIPs.

Justin's travel router uses 10.10.1.100/24 with 10.10.1.1 being the gateway.

Each node will get a persistent volume so installations and reboots work on the machines.
Persistent volumes are generated dynamically from CDI importer pods.
When a new datavolume is requested for a VM the CDI will create an importer job to seed the data into the PV and expand it to the requested size.

Workshop clusters should set a static IP address using a patch.
VMs use `/dev/vda` for install disk.

```
machine:
  network:
    interfaces:
      - interface: eth0
        addresses:
          - 10.10.1.150/22
        routes:
          - network: 0.0.0.0/0
            gateway: 10.10.1.1
            metric: 1024
        mtu: 1500
  install:
    disk: /dev/vda
```

## Mirror

The mirror is an nginx pod + service + pv to host files internal to the cluster for use.
This is where you should put raw volumes you download from the image factory using something like:

```bash
k cp nocloud-amd64.raw.xz nginx-deployment-67c67586bf-gkrqd:/usr/share/nginx/html/
```

The file would then be available inside the cluster at `http://mirror/nocloud-amd64.raw.xz`

## KubeVirt manager

If you want to use the web interface for KubeVirt manager you will need to port forward the traffic to access the service.

```bash
k port-forward -n kubevirt-manager svc/kubevirt-manager 42277:8080
```

Now open your browser to localhost:42277

## Registry pull through cache

TBD
