# ACM SNO

## Storage

### Prerequisites
* LVM Storage Operator
* Openshift Data Foundation (ODF) Operator

### LVM Storage
```yaml
apiVersion: lvm.topolvm.io/v1alpha1
kind: LVMCluster
  name: local-storage-lvmcluster
  namespace: openshift-storage
spec:
  storage:
    deviceClasses:
      - default: true
        deviceSelector:
          paths:
            - /dev/nvme0n1
        fstype: xfs
        name: local-storage
        thinPoolConfig:
          chunkSizeCalculationPolicy: Static
          metadataSizeCalculationPolicy: Host
          name: thin-pool-1
          overprovisionRatio: 10
          sizePercent: 90
```

### ODF StorageCluster for ObjectStorage
```yaml
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
```

## Observability

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
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: thanos-object-storage-obc
  namespace: open-cluster-management-observability
spec:
  bucketName: thanos-object-storage-bucket
  storageClassName: openshift-storage.noobaa.io
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

```yaml
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
```