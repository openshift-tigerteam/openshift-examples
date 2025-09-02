### Required Hardware

Below is a list of the recommended hardware for a basic OpenShift install. Consider these minimum values - the more the better. Disks must be [SSD/NVME](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/etcd/index#recommended-etcd-practices_etcd-practices). 

| Hosts (7 total)                   | CPU   | Memory    | Install Disk  |
| ---                               | ---   | ---       | ---           |
| bastion                           | 4     | 16 GB     | 60 GB         |
| openshift-control-plane-{1,2,3}   | 16    | 32 GB     | 120 GB        |
| openshift-worker-{1,2,3}          | 16    | 64 GB     | 120 GB        |

This also assumes your storage is handled elsewhere is already in place. If ODF will be used for storage, then worker machines would need additional CPU, RAM and an extra SSD/NVME drive in at least three of the worker machines to form a storage cluster. 

There is an ability to run the control plane nodes as schedulable, making them effectively worker nodes as well. This is discouraged unless required by the environment constraints, such as testing to run edge systems. 

> Run etcd on a block device that can write at least 50 IOPS of 8KB sequentially, including fdatasync, in under 10ms.
> [Consider moving etcd to it's own disk](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/etcd/index#move-etcd-different-disk_etcd-performance)