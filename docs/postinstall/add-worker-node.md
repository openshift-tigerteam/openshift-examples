# Adding a Worker Node to an Existing Cluster

[Back to Main](postinstall.md)

## Adding a Worker Node to an existing bare-metal on-premise OpenShift cluster

This guide describes the steps to add a new worker node to an existing OpenShift cluster. This example assumes you have a bare-metal on-premise OpenShift cluster and you want to add an additional worker node.

### Create the YAML configuration file

Create a file named `nodes-config.yaml` with the following content. Modify the values to match your environment. This shuld look very familiar as it is similar to the install-config.yaml file used during installation. If you are readding a node to an existing cluster, you can copy the install-config.yaml file and modify it as needed.

```yaml
hosts:
- hostname: extra-worker-1
  rootDeviceHints:
   deviceName: /dev/sda
  interfaces:
   - macAddress: 00:00:00:00:00:00
     name: eth0
  networkConfig:
   interfaces:
     - name: eth0
       type: ethernet
       state: up
       mac-address: 00:00:00:00:00:00
       ipv4:
         enabled: true
         address:
           - ip: 192.168.122.2
             prefix-length: 23
         dhcp: false
```
> [YAML file parameters](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/nodes/index#adding-node-iso-yaml-config_adding-node-iso)

<details>
<summary>nodes-config.yaml with bond0</summary>
<p>This example includes two nics and a bond.</p>
```yaml
{% include-markdown "postinstall/nodes-config-bond.yaml" %}
```
</details>

### Create the ISO image

Run the following command to create the ISO image. Modify the parameters to match your environment.
```bash
oc adm node-image create nodes-config.yaml --dir=add --registry-config=/path/to/pull-secret.txt 
```
> [Command flag options](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/nodes/index#adding-node-iso-flags-config_adding-node-iso)

Verify that a new `node.<arch>.iso` file is present in the asset directory. The asset directory is your current directory, unless you specified a different one when creating the ISO image.  

Boot the node with the generated ISO image.  

### Monitor the progress
```bash
oc adm node-image monitor --ip-addresses <ip_addresses>
```
> `<ip_addresses>` is a comma-separated list of IP addresses assigned to the new nodes.

### Approve any CSRs that are pending
```bash
oc get csr
oc get csr | awk '{print $1}' | grep -v NAME | xargs oc adm certificate approve
```

### Documentation Links
* [Adding worker node to an on-premise cluster](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/nodes/index#adding-node-iso)
