# Networking

## Overview

From a linux configuration perspective, remember that: 

* There are physical connections (`type: ethernet`)
* Those can be bonded (bond)
    * Bonds have a mode of `802.3ad` (LACP), `balance-xor`, or `active-backup`. 
* Vlans can have a `base-iface` of an existing `ethernet` or `bond`
    * You can have multiple vlans from a trunked `base-iface`

### Typical OpenShift Production Setup

* 7 NICs
* 3 Bonds - LACP
    * bond0 - management bond for cluster traffic
    * bond1 - data bond for pod network
    * bond2 - storage network
* 1 BMC interface

## NodeNetworkConfigurationPolicy Examples

This is just for setting up your bonds. 
<details>
<summary>2-eth-bond-lacp-vlan</summary>
```yaml
{% include-markdown "networking/2-eth-bond-lacp-vlan.yaml" %}
```
</details>

Another example for setting up your bonds with `active-backup`. 
<details>
<summary>2-eth-bond-active-backup-vlan</summary>
```yaml
{% include-markdown "networking/2-eth-bond-active-backup-vlan.yaml" %}
```
</details>

This is for when you want to create a trunk all the way to the OpenShift SDN.
<details>
<summary>ovs-bridge-trunk-nncp</summary>
<p>This example assumes you have an existing `bond1` on each of your worker nodes. One of the good things about using bonded connections is that if your device names are not consistent across your hardware, you have the ability to obfuscate the differences and name the bonds all the same, thus your additional configurations can be applied to larger groupings of hardware. </p>
```yaml
{% include-markdown "networking/ovs-bridge-trunk-nncp.yaml" %}
```
</details>

## ClusterUserDefinedNetwork

Once you have your NNCPs in place and your underlays are created, now you need to take it that last mile to the namespaces. 

<details>
<summary>cudn-with-ipam</summary>
<p>This example assumes you have an existing ovn bridge mapping for a localnet connection called localnet-bridge-trunk.</p>
```yaml
{% include-markdown "networking/cudn-with-ipam.yaml" %}
```
</details>

<details>
<summary>cudn-no-ipam</summary>
<p>This example assumes you have an existing ovn bridge mapping for a localnet connection called localnet-bridge-trunk.</p>
```yaml
{% include-markdown "networking/cudn-no-ipam.yaml" %}
```
</details>