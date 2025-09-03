# OpenShift Virtualization

## Prerequisites

* Cluster worker nodes have been installed on bare metal
* Hardware [requirements](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/virtualization/installing#virt-hardware-os-requirements_preparing-cluster-for-virt) have been met
* Storage has been [configured](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/virtualization/installing#storage-requirements_preparing-cluster-for-virt) for the cluster
    * A default OpenShift Virtualization or OpenShift Container Platform storage class has been created
      * To mark a storage class as the default for virtualization workloads, set the annotation `storageclass.kubevirt.io/is-default-virt-class` to `true`. 
    * Don't forget to [read](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/virtualization/installing#virt-about-storage-volumes-for-vm-disks_preparing-cluster-for-virt) about the need for RWX
* NMState Operator has been installed
    * All [networking setup](../networking/networking.md) in terms of underlay networks have been created
    * Optional, but recommended, [dedicated network for live migration](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/virtualization/networking#virt-dedicated-network-live-migration)

## Install

* Install the [Openshift Virtualization Operator](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/virtualization/installing#virt-installing-virt-operator_installing-virt) via the web console


### Example cloud-init for static ip

```yaml
networkData: |
  version: 2
  ethernets:
    eth1:
      dhcp4: no
      addresses:
        - 10.37.0.50/24
      gateway4: 10.37.0.1
      nameservers:
        addresses:
          - 10.3.0.3
          - 9.9.9.9
      dhcp6: no
      accept-ra: false
userData: |
  #cloud-config
  user: cloud-user
  password: Pass123!
  chpasswd:
    expire: false
```