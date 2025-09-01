### Required Hardware

Below is a list of the recommended hardware for a basic OpenShift install. Consider these minimum values - the more the better. Disks must be SSD/NVME. 

| Hosts (7 total)                   | CPU   | Memory    | Disk      |
| ---                               | ---   | ---       | ---       |
| bastion                           | 4     | 16 GB     | 60 GB     |
| openshift-control-plane-{1,2,3}   | 16    | 32 GB     | 120 GB    |
| openshift-worker-{1,2,3}          | 16    | 64 GB     | 120 GB    |

This also assumes youtr storage is handled elsewhere is already in place. If ODF will be used for storage, then worker machines would need additional CPU, RAM and an extra SSD/NVME drive in at least three of the worker machines to form a storage cluster. 