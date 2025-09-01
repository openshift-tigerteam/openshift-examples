# Networking

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

<details>
<summary>2-eth-bond-lacp-vlan</summary>
```yaml
{% include-markdown "networking/2-eth-bond-lacp-vlan.yaml" %}
```
</details>

<details>
<summary>2-eth-bond-active-backup-vlan</summary>
```yaml
{% include-markdown "networking/2-eth-bond-active-backup-vlan.yaml" %}
```
</details>