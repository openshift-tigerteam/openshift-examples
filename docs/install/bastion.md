The bastion host serves multiple purposes. First, all of our install work is done here. Second, it's used to validate the environment prior to installation. Third, it provides additional tools for installation such as a web server environment for the ISO images. 

### Install the OS

* Download [Red Hat Enterprise Linux 9.x Binary DVD](https://access.redhat.com/downloads/content/rhel)
* Boot host from ISO and perform install as `Server with GUI`
* Make sure to enable SSH for user
* Reboot and SSH into bastion host as administrative user

### Register the Host

To be able to install necessary packages, you need to register the host with your Red Hat ID and password.   
```shell
sudo subscription-manager register # Enter username/password
sudo subscription-manager repos --enable=rhel-9-for-x86_64-baseos-rpms
sudo subscription-manager repos --enable=rhel-9-for-x86_64-appstream-rpms
sudo dnf update -y 
sudo reboot
```

### Download Required Tools

Install the required tools.   
```shell
OCP_VERSION={{ ocp_version }}
wget "https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable-${OCP_VERSION}/openshift-install-linux.tar.gz" -P /tmp
sudo tar -xvzf /tmp/openshift-install-linux.tar.gz -C /usr/local/bin
wget "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable-${OCP_VERSION}/openshift-client-linux.tar.gz" -P /tmp
sudo tar -xvzf /tmp/openshift-client-linux.tar.gz -C /usr/local/bin
rm /tmp/openshift-install-linux.tar.gz /tmp/openshift-client-linux.tar.gz -y
sudo dnf install nmstate git podman 
```

Check for the availability of the required tools.    
```shell
openshift-install version
oc version
nmstatectl -V
git -v
podman --version
```

### Get the Pull Secret

Pull Secret is available at https://console.redhat.com/openshift/install/pull-secret
Download to ~/.pull-secret

### Create SSH Key
```shell
ssh-keygen -t ed25519 -f ~/.ssh/ocp_ed25519
```

### Open Port 8080 for iso-http
```shell
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

### Tools for Environment Validation 

```shell
ping registry.redhat.io        # ICMP doesn’t always work, but try
curl -vk https://registry.redhat.io/v2/
dig registry.redhat.io +short
nslookup registry.redhat.io
podman login registry.redhat.io
```