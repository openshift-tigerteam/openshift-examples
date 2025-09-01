# OpenShift Virtualization


### Example cloud-init for static ip

```yaml
networkData: |
  version: 2
  ethernets:
    eth1:
      dhcp4: no
      addresses:
        - 10.37.0.50/24
      gateway4: 10.37.0.1
      nameservers:
        addresses:
          - 10.3.0.3
          - 9.9.9.9
      dhcp6: no
      accept-ra: false
userData: |
  #cloud-config
  user: cloud-user
  password: Pass123!
  chpasswd:
    expire: false
```