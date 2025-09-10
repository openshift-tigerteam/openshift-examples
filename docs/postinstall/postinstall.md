# Post Install

## Before Storage

Install the Operators:

* Kubernetes NMState Operator
* Node Feature Discovery
* Workload Availability for Red Hat OpenShift
    * Node Health Check Operator
    * Self Node Remediation Operator
    * Kube Descheduler Operator
* Cert-manager Operator for Red Hat OpenShift

### Networking

Add additional networking if required. See [Networking](/docs/networking/networking.md) documentation.

## Storage

Install storage...

## After Storage

Install the Operators: 

* [OpenShift Virtualization](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/virtualization/installing#installing-virt)
* [Configure Registry](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/registry/setting-up-and-configuring-the-registry#configuring-registry-storage-baremetal)


## Optional

* [Add Worker Node](add-worker-node.md)  
* [Rotate SSH Keys](rotate-ssh-keys.md)
