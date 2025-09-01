# Install

These install documents are focused on on-premise installations on bare metal. There are other install methods. If you can use the [Assisted Installer](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/installing_on-premise_with_assisted_installer/index#using-the-assisted-installer_installing-on-prem-assisted), do so. 

You will also need an entitled Red Hat account. 

Please read the entire page prior to starting. 

## Steps

* Acquire the Hardware
* Gather the Information
* Update the Network
* Build the Bastion Host
* Create the Configurations
* Generate the ISO
* Boot the Machines
* Install the Cluster
* Validate the Install

## Acquire the Hardware
{% include-markdown "install/baremetal_hardware.md" %}

## Gather the Information

### Cluster Values
Below are the values an enterprise typically has to gather or create for installing OpenShift.

|                               | Example Value             | Description                                                                       |
| ---                           | ---                       | ---                                                                               |
| **Cluster Name**              | poc                       | Name of the cluster                                                               |
| **Base Domain**               | ocp.basedomain.com        | Name of the domain                                                                |
| **Machine Subnet**            | 10.1.0.0/24 (vlan - 123)  | Subnet and vlan for all machines/VIPs in cluster (can be much smaller)            |
| **Pod Subnet**                | 10.128.0.0/14             | Subnet for pod SDN                                                                |
| **Pod Subnet - Host Prefix**  | 23                        | Host prefix for Subnet for pod SDN                                                |
| **Service Subnet**            | 172.30.0.0/16             | Subnet for service SDN                                                            |
| **API VIP**                   | 10.1.0.9                  | VIP for the MetalLB API Endpoint. **This must be inside the machine subnet**      |
| **Ingress VIP**               | 10.1.0.10                 | VIP for the MetalLB Ingress Endpoint. **This must be inside the machine subnet**  |
| **DNS**                       | dns1.basedomain.com,etc   | IP or hostname for the DNS hosts                                                  |
| **NTP**                       | ntp.basedomain.com,etc    | IP or hostname for the NTP hosts                                                  |

#### SDN Subnet Overlaps

The Pod Subnet and Service Subnet are run in the software defined network (SDN) of the cluster. If those values conflict with existing subnets **and** your pods in the cluster will want to route to those outside services with conflicting IPs, you will need to provide different subnets. It's much easier to ensure they are unique.  

#### Pod Subnet and Host Prefix Explanation

The pod subnet in this example is huge. If we are just doing a small cluster, think like 500 pods per node. 6 nodes with 500 pods is 3000 ips. You could use a `/20` and then each node would have a `/23` for a total of 8 nodes. ex. 10.128.0.0/20 with hostPrefix: 23. 

```
10.128.0.0/23    (10.128.0.0 – 10.128.1.255)
10.128.2.0/23    (10.128.2.0 – 10.128.3.255)
10.128.4.0/23    (10.128.4.0 – 10.128.5.255)
10.128.6.0/23    (10.128.6.0 – 10.128.7.255)
10.128.8.0/23    (10.128.8.0 – 10.128.9.255)
10.128.10.0/23   (10.128.10.0 – 10.128.11.255)
10.128.12.0/23   (10.128.12.0 – 10.128.13.255)
10.128.14.0/23   (10.128.14.0 – 10.128.15.255)
```

> [Pod Subnet - Host Prefix](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/installing_on_bare_metal/index#CO63-10)  
> [CIDR Subnet Reference](https://docs.netgate.com/pfsense/en/latest/network/cidr.html#understanding-cidr-subnet-mask-notation)

### Machine Information Required

Typically, machines will have more than one NIC and these will be setup in a bond. Please collect the interface names and MAC addresses for ALL NICS and the install disk location on the machines. You provide the hostnames, IPs. IPs need to be located in the machine configuration subnet used on the install. Below is an example table of values needed for collection. 

| Hostname                  | Interface | MAC Address       | Bond IP   | Disk Hint |
| ---                       | ---       | ---               | ---       | ---       |
| bastion                   | eno1      | A0-B1-C2-D3-E4-E1 | 10.1.0.5  | /dev/sda  |
| openshift-control-plane-1 | eno1      | A0-B1-C2-D3-E4-E1 | 10.1.0.11 | /dev/sda  |
|                           | eno2      | A0-B1-C2-D3-E4-E2 |           |
| openshift-control-plane-2 | eno1      | A0-B1-C2-D3-E4-E3 | 10.1.0.12 | /dev/sda  |
|                           | eno2      | A0-B1-C2-D3-E4-E4 |           |
| openshift-control-plane-3 | eno1      | A0-B1-C2-D3-E4-E5 | 10.1.0.13 | /dev/sda  |
|                           | eno2      | A0-B1-C2-D3-E4-E6 |           |
| openshift-worker-1        | eno1      | A0-B1-C2-D3-E4-F1 | 10.1.0.21 | /dev/sda  |
|                           | eno2      | A0-B1-C2-D3-E4-F2 |           |
| openshift-worker-2        | eno1      | A0-B1-C2-D3-E4-F3 | 10.1.0.22 | /dev/sda  |
|                           | eno2      | A0-B1-C2-D3-E4-F4 |           |
| openshift-worker-3        | eno1      | A0-B1-C2-D3-E4-F5 | 10.1.0.23 | /dev/sda  |
|                           | eno2      | A0-B1-C2-D3-E4-F6 |           |

The IPs in the example table represent a bonded IP or if the cluster does not use bonding, that IP that connects the machine to the machine network. Notice all machine IPs are inside of the machine subnet of 10.1.0.0/24. 

## Update the Network
### Firewall and Networking Requirements

Prior to the install, you must open the firewall to connect to Red Hat's servers and ports between the machines.

* [Configuring Firewall](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/installation_configuration/configuring-firewall#configuring-firewall_configuring-firewall)
* [Network Connectivity Requirements](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/installing_on_bare_metal/index#installation-network-connectivity-user-infra_installing-bare-metal)
* [Ensuring required ports are open](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/installing_on_bare_metal/index#network-requirements-ensuring-required-ports-are-open_ipi-install-prerequisites)

### Create DNS Entries

Create the following A records in your DNS based on the values from above. 

| A Record                          | IP Address  | Description                         | 
| ---                               | ---         | ---                                 |
| api.poc.ocp.basedomain.com        | 10.1.0.9    | Virtual IP for the API endpoint     |
| api-int.poc.ocp.basedomain.com    | 10.1.0.9    | Virtual IP for the API endpoint     |
| *.apps.poc.ocp.basedomain.com     | 10.1.0.10   | Virtual IP for the ingress endpoint |

You can validate the DNS using dig

```shell
dig +noall +answer @<nameserver_ip> api.<cluster_name>.<base_domain>
dig +noall +answer @<nameserver_ip> test.apps.<cluster_name>.<base_domain>
```

> [DNS requirements](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/installing_on_bare_metal/index#network-requirements-dns_ipi-install-prerequisites)  
> [Validating DNS resolution for user-provisioned infrastructure](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/installing_on_bare_metal/index#installation-user-provisioned-validating-dns_installing-bare-metal-network-customizations)

#### Optional Helpful DNS Entries

| A Record                                          | IP Address  | Description         | 
| ---                                               | ---         | ---                 |
| bastion.poc.ocp.basedomain.com                    | 10.1.0.4    | IP for the bastion  |
| openshift-control-plane-1.poc.ocp.basedomain.com  | 10.1.0.11    | IP for cp1         |
| openshift-control-plane-2.poc.ocp.basedomain.com  | 10.1.0.12    | IP for cp2         |
| openshift-control-plane-3.poc.ocp.basedomain.com  | 10.1.0.13    | IP for cp3         |
| openshift-worker-1.poc.ocp.basedomain.com         | 10.1.0.21    | IP for w1          |
| openshift-worker-2.poc.ocp.basedomain.com         | 10.1.0.22    | IP for w2          |
| openshift-worker-3.poc.ocp.basedomain.com         | 10.1.0.23    | IP for w3          |

## Build the Bastion Host

{% include-markdown "install/bastion.md" %}

## Create the Configurations
{% include-markdown "install/config.md" %}

## Generate the ISO

## Boot the Machines

## Install the Cluster

## Validate the Install