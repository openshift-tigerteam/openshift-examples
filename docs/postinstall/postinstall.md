# Post Install

## Before Storage

Install the Operators

* Kubernetes NMState Operator

### Networking

Add additional networking if required. See [Networking](/docs/networking/networking.md) documentation.

## Storage

<Install Storage>

## After Storage

Install the Operators

* OpenShift Virtualization
* Node Feature Discovery
* Workload Availability for Red Hat OpenShift
* Node Health Check Operator
  * Self Node Remediation Operator
  * Fence Agents Remediation Operator
  * Kube Descheduler Operator
* Configure Registry
* Cert-manager Operator for Red Hat OpenShift

## Optional

[Rotate SSH Keys](rotate-ssh-keys.md)
