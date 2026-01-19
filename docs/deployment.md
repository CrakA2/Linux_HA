# Deployment Workflows

## HA Cluster Setup

```bash
# 1. Install components
apt-get install corosync pacemaker pcs

# 2. Generate keys
corosync-keygen

# 3. Configure Corosync
vim /etc/corosync/corosync.conf

# 4. Start cluster
systemctl enable corosync pacemaker pcsd
systemctl start corosync pacemaker pcsd

# 5. Configure Pacemaker
pcs cluster setup --name mycluster node1 node2 node3
pcs cluster start --all
```

## Virtualization Setup

```bash
# 1. Install packages
apt-get install qemu-kvm libvirt-daemon-system \
    libvirt-clients virt-manager openvswitch-switch

# 2. Enable services
systemctl enable libvirtd
systemctl start libvirtd

# 3. Configure networking
systemctl enable openvswitch
systemctl start openvswitch

# 4. Create VM
virt-install --name vm1 --memory 2048 --vcpus 2 \
    --disk path=/var/lib/libvirt/images/vm1.qcow2,size=20 \
    --network network=default
```

## CEPH HCI Setup

```bash
# 1. Install cephadm
curl --silent --remote-name --location \
    https://download.ceph.com/rpm-18.2.1/el9/noarch/cephadm \
    -o cephadm
chmod +x cephadm

# 2. Bootstrap cluster
./cephadm bootstrap --mon-ip <mon-ip>

# 3. Add storage
ceph orch apply osd --all-available-devices

# 4. Verify cluster
ceph -s
ceph status
```
