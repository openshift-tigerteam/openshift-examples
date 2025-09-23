# Configuring the OpenShift Registry

[Back](../postinstall.md)

The OpenShift Container Platform internal image registry is deployed by default as part of the installation. By default, the registry uses ephemeral storage, which means that images stored in the registry are lost if the registry pod is rescheduled. To provide persistent storage for the registry, you can configure it to use a Persistent Volume Claim (PVC).

```shell
cat <<EOF | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: registry-storage-pvc
 namespace: openshift-image-registry
spec:
 accessModes:
 - ReadWriteMany
 resources:
   requests:
     storage: 100Gi
 storageClassName: ocs-storagecluster-cephfs
EOF
```

Once the PVC is created, you can patch the image registry configuration to use the PVC for storage.  
```shell
oc patch config.image/cluster -p '{"spec":{"managementState":"Managed","replicas":2,"storage":{"managementState":"Unmanaged","pvc":{"claim":"registry-storage-pvc"}}}}' --type=merge
```

### Documentation

[Configuring the Registry](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/registry/setting-up-and-configuring-the-registry)  
[Configuring with ODF](https://docs.redhat.com/en/documentation/red_hat_openshift_data_foundation/latest/html-single/managing_and_allocating_storage_resources/index#configuring-image-registry-to-use-openshift-data-foundation_rhodf)  