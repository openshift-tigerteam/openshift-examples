The typical agent based install requires two configuration files. 

* `install-config.yaml`
* `agent-config.yaml`

The `install-config.yaml` contains all the cluster level information while the `agent-config.yaml` provides host level configurations. 

> You can accomplish all of this with bastion tools like `vi`. It is highly recommended to use tools like Visual Studio Code (vscode) to help with yaml formatting. 

### Create the Working Directory

You are going to want to create a working directory and initialize it into a git repository. 

```shell 
mkdir -p ocp && cd ocp
git init 
echo "install/" > .gitignore
echo "# POC install notes" > notes.md
touch install-config.yaml
touch agent-config.yaml
git add -A
git commit -m "repo initialized"
```

### Create the Install Config

Here is an example of the `install-config.yaml` contents.
```yaml
apiVersion: v1
baseDomain: ocp.basedomain.com
metadata:
  name: poc
controlPlane:
  name: master
  architecture: amd64
  hyperthreading: Enabled
  replicas: 3
compute:
- name: worker
  architecture: amd64
  hyperthreading: Enabled
  replicas: 3
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.1.0.0/24
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  baremetal:
    apiVIP: 10.1.0.9
    ingressVIP: 10.1.0.10
    additionalNTPServers: 
      - 0.us.pool.ntp.org
      - 1.us.pool.ntp.org
pullSecret: 'value from ~/.pull_secret'
sshKey: 'value from ~/.ssh/ocp_ed25519.pub'
```

If your environment requires a proxy to access the internet then copy, modify and append to the end of the `install-config.yaml`
```yaml 
proxy:
  httpProxy: http://user:password@proxy.example.com:3128
  httpsProxy: http://user:password@proxy.example.com:3128
  noProxy: poc.ocp.basedomain.com,10.1.0.0/24,10.128.0.0/14,172.30.0.0/16,api.poc.ocp.basedomain.com,api-int.poc.ocp.basedomain.com,*.apps.poc.ocp.basedomain.com
```

### Create the Agent Config

The agent-config.yaml is basically a list of hosts' information. You'll create a list item for each of the hosts. Here's an example of a host with two ethernet connections bonded together in an LACP bond. 

```yaml
apiVersion: v1alpha1
kind: AgentConfig
metadata:
  name: poc
rendezvousIP: 10.1.0.11                     # IP of the first control-plane1 host
additionalNtpSources:
  - 0.us.pool.ntp.org
  - 1.us.pool.ntp.org
hosts:
  - hostname: openshift-control-plane-1     # hostname
    role: master                            # master/worker
    rootDeviceHints:
      deviceName: "/dev/sda"                # Disk Hint
    interfaces:
      - name: eno1                          # Interface Name 1
        macAddress: A1:B2:3C:4D:1E:11       # Mac Address 1
      - name: eno2                          # Interface Name 2
        macAddress: A1:B2:3C:4D:2E:11       # Mac Address 2
    networkConfig:
      interfaces:
        - name: eno1                        # Interface Name 1
          type: ethernet
          state: up
          mac-address: A1:B2:3C:4D:1E:11    # Mac Address 1
          ipv4:
            enabled: false
          ipv6:
            enabled: false
        - name: eno2                        # Interface Name 2
          type: ethernet
          state: up
          mac-address: A1:B2:3C:4D:2E:11    # Mac Address 2
          ipv4:
            enabled: false
          ipv6:
            enabled: false
        - name: bond0
          type: bond
          state: up
          ipv4:
            enabled: true
            address:
              - ip: 10.1.0.11               # Host IP inside the machine subnet
                prefix-length: 24
            dhcp: false
          link-aggregation:
            mode: 802.3ad                   # LACP
            port:
              - eno1
              - eno2
            options:
              miimon: "100"
              lacp_rate: fast
          ipv6:
            enabled: false
      dns-resolver:
        config:
          server:
            - dns1.basedomain.com           # DNS 1
            - dns2.basedomain.com           # DNS 2
      routes:
        config:
          - destination: 0.0.0.0/0
            next-hop-address: 10.1.0.1
            next-hop-interface: bond0
            table-id: 254
```

> Notice the inconsistent labels and spellings.  
>   * `macAddress` in the interfaces stanza, but `mac-address` in the networkConfig stanza.    
>   * `additionalNtpSources` is used here, but it's `additionalNTPServers` in the `install-config.yaml`

[Docs: Sample Config Bonds Vlans](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/installing_an_on-premise_cluster_with_the_agent-based_installer/preparing-to-install-with-agent-based-installer#agent-install-sample-config-bonds-vlans_preparing-to-install-with-agent-based-installer)