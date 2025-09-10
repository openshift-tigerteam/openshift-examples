# Adding a Worker Node to an Existing Cluster

[Back to Main](postinstall.md)

## Adding a Worker Node to an existing bare-metal on-premise OpenShift cluster

This guide describes the steps to add a new worker node to an existing OpenShift cluster. This example assumes you have a bare-metal on-premise OpenShift cluster and you want to add an additional worker node.

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

```bash
oc adm node-image create nodes-config.yaml --dir=add --registry-config=/path/to/pull-secret.txt 
```
> [Command flag options](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/nodes/index#adding-node-iso-flags-config_adding-node-iso)

Monitor the progress
```bash
oc adm node-image monitor --ip-addresses <ip_addresses>
```

Approve any CSRs that are pending
```bash
oc get csr
oc get csr | awk '{print $1}' | grep -v NAME | xargs oc adm certificate approve
```


### Documentation Links
* [Adding worker node to an on-premise cluster](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html-single/nodes/index#adding-node-iso)
