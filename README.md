
# Ceph Installation RHCS 5

## Requirements
* 4 Hosts with HDD type 3 disk attached each
* Red Hat subscription
* One client Hosts
* 8 GB RAM 
* 4 core cpu
* RHEL 8.6 OS

### Note: All the work is done through the root user.
## Installation

* First install the RHEL 8.6 OS on all the hosts including the client hosts. Then run the following command to register the system for the subscription.
```bash
subscription-manager register
```
It will ask you for the id and password enter that.

* Edit hosts file at /etc/hosts. Enter all the hosts and client in FQDN system.
```bash
vim /etc/hosts
10.25.23.212    host1.example.com   host1
10.25.23.212    host2.example.com   host2   
10.25.23.212    host3.example.com   host3
10.230.25.21    client.example.com   client
```
* you can copy the hosts file to other hosts too.
```bash
scp /etc/hosts host1:/etc/hosts
scp /etc/hosts host2:/etc/hosts
scp /etc/hosts host3:/etc/hosts
scp /etc/hosts client:/etc/hosts

* Enable the following repository
```bash
subscription-manager repos --enable rhceph-5-tools-for-rhel-8-x86_64-rpms
subscription-manager repos --enable ansible-2.9-for-rhel-8-x86_64-rpms
```
* Generate a new ssh key and copy it to all nodes.
```bash
ssh-keygen
ssh-copy-id host1
ssh-copy-id host2
ssh-copy-id host3
ssh-copy-id client
```

* install the cepadm package
```bash
dnf install cephadm-ansible -y
```
* Goto the "/usr/share/cephadm-ansible" directory and create one hosts file where you need to mention about the host information.
```bash
host02
host03

[admin]
host01

[clients]
client
```

* Then run the preflight checking playbook after mentioning the host
```bash
ansible-playbook -i hosts cephadm-preflight.yml --extra-vars "ceph_origin=rhcs"
```

* Now crete one sppec which contains the configuration of three important ceph daemon - monitors (mons), managers (mgr), and osds. Following is an sample spec file which contains 3 hosts and one client with three mons, mgr and 9 osds.
```bash
vim specs.yaml
```
```bash
service_type: host
addr: host1
hostname: host1
---
service_type: host
addr: host2
hostname: host2
---
service_type: host
addr: host3
hostname: host3
---
service_type: host
addr: host4
hostname: host4
---
service_type: mgr
placement:
  host_pattern: "host[1-3]"
---
service_type: mon
placement:
  host_pattern: "host[1-3]"
---
service_type: osd
service_id: my_osds
placement:
  host_pattern: "host[1-4]"
data_devices:
  all: true
```

* Create json file to store registry credentials
```bash
vim mylogin.json
{
 "url":"registry.redhat.io",
 "username":"myuser1",
 "password":"mypassword1"
}
```

* Now you are ready to bootstrap cluster. Run the following command if you are following this guide.
```bash
cephadm bootstrap --apply-spec boot.yaml --mon-ip 10.10.128.68 --registry-json mylogin.json
```
* Try this command if you have not copied ssh key using "ssh-copy-id" command.
```bash
cephadm bootstrap --apply-spec boot.yaml --mon-ip 10.10.128.68 --ssh-private-key /home/ceph/.ssh/id_rsa --ssh-public-key /home/ceph/.ssh/id_rsa.pub --registry-json mylogin.json
```

* If you share different share different ssh key and register login name and password manually, then you can run following command.
```bash
cephadm bootstrap --apply-spec boot.yaml --mon-ip 10.10.128.68 --ssh-private-key /home/ceph/.ssh/id_rsa --ssh-public-key /home/ceph/.ssh/id_rsa.pub --registry-url registry.redhat.io --registry-username myuser1 --registry-password mypassword1
```
### Note: Here "mon" ip your bootstrap cluster Ip
