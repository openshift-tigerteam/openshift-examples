# Creating the Hub Cluster

## Hardware Requirements
* Single box. Recommend 24 cores, 64 GB RAM and two disks - os disk 120 GB, data disk 2TB
* Can be bare metal or virtualized. No OpenShift Virtualization here. 

## Install SNO
* Highly recommend assisted installer for this. 

## Install Storage

These examples are in leiu of existing storage. 

### Install Storage Operators
* LVM Storage Operator
* Openshift Data Foundation (ODF) Operator  

Label the node as a storage node  
```shell
oc label node <node-name> cluster.ocs.openshift.io/openshift-storage=
```

### Create LVM Storage

Below is an example. Change out the path for your data disk. 
```shell
cat <<EOF | oc apply -f -
apiVersion: lvm.topolvm.io/v1alpha1
kind: LVMCluster
metadata:
  name: local-storage-lvm-cluster
  namespace: openshift-storage
spec:
  storage:
    deviceClasses:
      - name: local-storage
        default: true
        fstype: xfs
        deviceSelector:
          paths:
            - /dev/nvme0n1
        thinPoolConfig:
          name: thin-pool-1
          sizePercent: 90
          overprovisionRatio: 10
          chunkSizeCalculationPolicy: Static
          metadataSizeCalculationPolicy: Host
EOF
```

### Create ODF StorageCluster for ObjectStorage

We are using ODF only for the ObjectStorage
```shell
cat <<EOF | oc apply -f -
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: mcog-storagecluster
  namespace: openshift-storage
spec:
  arbiter: {}
  encryption:
    keyRotation:
      schedule: '@weekly'
    kms: {}
  externalStorage: {}
  managedResources:
    cephObjectStoreUsers: {}
    cephCluster: {}
    cephBlockPools: {}
    cephNonResilientPools: {}
    cephObjectStores: {}
    cephFilesystems: {}
    cephRBDMirror: {}
    cephToolbox: {}
    cephDashboard: {}
    cephConfig: {}
  multiCloudGateway:
    dbStorageClassName: lvms-local-storage
    reconcileStrategy: standalone
  resourceProfile: balanced
EOF
```

## Install MultiClusterObservability

* Storage is installed (above)

```shell
oc create namespace open-cluster-management-observability
DOCKER_CONFIG_JSON=`oc extract secret/pull-secret -n openshift-config --to=-`
oc create secret generic multiclusterhub-operator-pull-secret \
    -n open-cluster-management-observability \
    --from-literal=.dockerconfigjson="$DOCKER_CONFIG_JSON" \
    --type=kubernetes.io/dockerconfigjson
```

```yaml
cat <<EOF | oc apply -f -
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: thanos-object-storage-obc
  namespace: open-cluster-management-observability
spec:
  bucketName: thanos-object-storage-bucket
  storageClassName: openshift-storage.noobaa.io
EOF
```

```shell
ACCESS_KEY=$(oc get secret thanos-object-storage-obc -n open-cluster-management-observability -o jsonpath='{.data.AWS_ACCESS_KEY_ID}' | base64 -d)
SECRET_KEY=$(oc get secret thanos-object-storage-obc -n open-cluster-management-observability -o jsonpath='{.data.AWS_SECRET_ACCESS_KEY}' | base64 -d)
```

```shell
cat <<EOF | oc apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: thanos-object-storage
  namespace: open-cluster-management-observability
type: Opaque
stringData:
  thanos.yaml: |
    type: s3
    config:
      bucket: thanos-object-storage-bucket
      endpoint: s3.openshift-storage.svc:443
      insecure: false
      access_key: $ACCESS_KEY
      secret_key: $SECRET_KEY
EOF
```

```shell
cat <<EOF | oc apply -f -
apiVersion: observability.open-cluster-management.io/v1beta2
kind: MultiClusterObservability
metadata:
  name: multi-cluster-observability
spec:
  enableDownsampling: true
  imagePullPolicy: Always
  imagePullSecret: multiclusterhub-operator-pull-secret
  observabilityAddonSpec:
    enableMetrics: true
    interval: 300
  storageConfig:
    alertmanagerStorageSize: 1Gi
    compactStorageSize: 100Gi
    metricObjectStorage:
      key: thanos.yaml
      name: thanos-object-storage
    receiveStorageSize: 100Gi
    ruleStorageSize: 1Gi
    storageClass: lvms-local-storage
    storeStorageSize: 10Gi
EOF
```

## Install ACM
* Install Advanced Cluster Management Operator

### Create InfraEnv

```shell
cat <<EOF | oc apply -f -
apiVersion: agent-install.openshift.io/v1beta1
kind: InfraEnv
metadata:
  name: my-infraenv
  namespace: <your-namespace>
spec:
  pullSecretRef:
    name: pull-secret
  sshAuthorizedKey: <your-ssh-public-key>
  clusterRef:
    name: <cluster-name>
    namespace: <your-namespace>
  nmStateConfigLabelSelector:
    matchLabels:
      infraenvs.agent-install.openshift.io: my-infraenv
EOF
```

### Adding Host Investory via Redfish

* Advanced Cluster Management for Kubernetes is installed
* Bare Metal Operator is installed
* Redfish credentials are available for the target host

```shell
oc create secret generic worker-0-bmc-secret \
  --from-literal=username=admin \
  --from-literal=password=your-bmc-password \
  -n <your-namespace>
```

```shell
echo -n "admin" | base64
echo -n "your-bmc-password" | base64  

cat <<EOF | oc apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: worker-0-bmc-secret
  namespace: <your-namespace>
type: Opaque
data:
  username: <base64-encoded-username>
  password: <base64-encoded-password>
EOF
```


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