# Adding Host Investory via Redfish

* Advanced Cluster Management for Kubernetes is installed
* Bare Metal Operator is installed
* Redfish credentials are available for the target host


```yaml
apiVersion: metal3.io/v1alpha1
kind: BareMetalHost
metadata:
  name: host-one
  namespace: openshift-machine-api
spec:
  online: false
  bootMACAddress: "aa:bb:cc:dd:ee:ff"
  bmc:
    address: redfish-protocol://<BMC-IP-address>/redfish/v1/
    credentialsName: "host-one-bmc-secret"
  hardwareProfile: "default"
  rootDeviceHints:
    hctl: "0:0:0:0"
```