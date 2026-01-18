# Comprehensive Cheatsheets

## Table of Contents
1. [Corosync Cheatsheet](#1-corosync-cheatsheet)
2. [Pacemaker Cheatsheet](#2-pacemaker-cheatsheet)
3. [QEMU/KVM Cheatsheet](#3-qemukvm-cheatsheet)
4. [Quick Reference Tables](#4-quick-reference-tables)
5. [Common Workflows](#5-common-workflows)
6. [Nifty Behaviors and Tips](#6-nifty-behaviors-and-tips)

---

## 1. Corosync Cheatsheet

### 1.1 Installation and Setup

```bash
# Installation
apt-get install corosync         # Debian/Ubuntu
yum install corosync             # RHEL/CentOS

# Generate authentication key
corosync-keygen

# Start and enable services
systemctl enable corosync
systemctl start corosync
```

### 1.2 Configuration Files

```bash
# Main configuration
/etc/corosync/corosync.conf

# Authentication key
/etc/corosync/authkey

# Log files
/var/log/corosync/corosync.log
/var/log/syslog                 # System log
```

### 1.3 Essential Commands

```bash
# Cluster status
corosync-cfgtool -s             # Show cluster status
corosync-cfgtool -a             # Show all nodes
corosync-cfgtool -l             # Show node list

# Quorum status
corosync-quorumtool -s          # Show quorum status
corosync-quorumtool -q          # Show quorum calculation
corosync-quorumtool -n          # Show node information

# Configuration database
corosync-cmapctl                 # Query configuration
corosync-cmapctl -b              # Reload configuration
corosync-cmapctl -t nodelist    # Show node list

# Service control
systemctl start corosync         # Start corosync
systemctl stop corosync          # Stop corosync
systemctl restart corosync       # Restart corosync
systemctl status corosync        # Check status
```

### 1.4 Configuration Parameters

```conf
totem {
    version: 2                    # TOTEM protocol version
    cluster_name: mycluster       # Cluster identifier
    transport: udpu               # Transport type (udp, udpu, knet)
    token: 10000                 # Token timeout (ms)
    token_retransmit: 500        # Token retransmit interval (ms)
    hold: 180                    # Hold timeout (ms)
    join: 60                     # Join timeout (ms)
    
    interface {
        ringnumber: 0
        bindnetaddr: 192.168.1.0
        mcastport: 5405
    }
}

nodelist {
    node {
        ring0_addr: 192.168.1.10
        ring1_addr: 192.168.2.10
        nodeid: 1
    }
}

quorum {
    provider: corosync_votequorum
    expected_votes: 3
    two_node: 0
}

logging {
    to_logfile: yes
    logfile: /var/log/corosync/corosync.log
    to_syslog: yes
    debug: off
}
```

### 1.5 Troubleshooting

```bash
# Check cluster membership
corosync-cfgtool -s

# Verify quorum
corosync-quorumtool -s

# Monitor logs
tail -f /var/log/corosync/corosync.log
journalctl -u corosync -f

# Check network connectivity
netstat -an | grep 5405

# Verify authkey
ls -l /etc/corosync/authkey

# Test configuration
corosync-cfgtool -k             # Kill corosync
corosync-cfgtool -r             # Reload configuration
```

### 1.6 Common Patterns

```bash
# Add new node
1. Generate and distribute authkey
2. Add node to nodelist in corosync.conf
3. Restart corosync on all nodes

# Change transport
1. Update transport in totem section
2. Update interface configuration
3. Restart corosync on all nodes

# Enable debug logging
1. Set debug: on in logging section
2. Set log level to debug
3. Restart corosync
```

---

## 2. Pacemaker Cheatsheet

### 2.1 Installation and Setup

```bash
# Installation
apt-get install pacemaker pcs      # Debian/Ubuntu
yum install pacemaker pcs          # RHEL/CentOS

# Start and enable services
systemctl enable pacemaker
systemctl start pacemaker
systemctl enable pcsd
systemctl start pcsd

# Authenticate nodes
pcs cluster auth node1 node2 node3 -u hacluster

# Create cluster
pcs cluster setup --name mycluster node1 node2 node3
pcs cluster start --all
pcs cluster enable --all
```

### 2.2 Essential Commands

```bash
# Cluster status
pcs status                        # Full cluster status
pcs status resources             # Resources only
pcs status nodes                # Nodes only
pcs status corosync             # Corosync status

# Resource management
pcs resource list               # List resources
pcs resource create             # Create resource
pcs resource start             # Start resource
pcs resource stop              # Stop resource
pcs resource move             # Move resource
pcs resource delete           # Delete resource

# Constraints
pcs constraint show            # Show constraints
pcs constraint order           # Add ordering
pcs constraint colocation      # Add colocation
pcs constraint location        # Add location

# Cluster properties
pcs property list             # List properties
pcs property set             # Set property
pcs property unset           # Unset property

# STONITH
pcs stonith list             # List STONITH devices
pcs stonith create           # Create STONITH
pcs stonith show             # Show STONITH config
```

### 2.3 Resource Management

```bash
# Create IP resource
pcs resource create vip ocf:heartbeat:IPaddr2 \
    ip=192.168.1.100 cidr_netmask=24 \
    op monitor interval=30s

# Create service resource
pcs resource create apache ocf:heartbeat:apache \
    configfile=/etc/apache2/apache2.conf \
    op monitor interval=30s

# Create group
pcs resource create web-group vip apache

# Create clone
pcs resource create haproxy-clone haproxy \
    clone

# Create master/slave
pcs resource create mysql-master mysql \
    master

# Resource operations
pcs resource start web-group
pcs resource stop web-group
pcs resource move web-group node2
pcs resource clear web-group

# Resource cleanup
pcs resource cleanup web-group
pcs resource refresh web-group
```

### 2.4 Constraints

```bash
# Ordering constraints
pcs constraint order start vip then apache
pcs constraint order stop apache then vip

# Colocation constraints
pcs constraint colocation add apache with vip
pcs constraint colocation add apache with vip score=100

# Location constraints
pcs constraint location web-group prefers node1=100
pcs constraint location web-group avoids node1=-INFINITY

# Resource stickiness
pcs resource update web-group \
    meta resource-stickiness=100

# Show constraints
pcs constraint show --full
pcs constraint order show
pcs constraint colocation show
pcs constraint location show
```

### 2.5 Cluster Properties

```bash
# Essential properties
pcs property set stonith-enabled=true
pcs property set no-quorum-policy=stop
pcs property set default-resource-stickiness=100
pcs property set symmetric-cluster=true

# Quorum policies
pcs property set no-quorum-policy=stop     # Stop resources
pcs property set no-quorum-policy=freeze    # Freeze resources
pcs property set no-quorum-policy=ignore   # Continue (dangerous)

# Resource defaults
pcs resource defaults resource-stickiness=100
pcs resource defaults migration-threshold=3
pcs resource defaults failure-timeout=120s

# Maintenance mode
pcs property set maintenance-mode=true
pcs property set maintenance-mode=false

# Display properties
pcs property list
pcs property config
```

### 2.6 STONITH Configuration

```bash
# IPMI STONITH
pcs stonith create ipmi-fence fence_ipmilan \
    ipaddr=192.168.1.200 login=admin \
    passwd=password pcmk_host_list="node1 node2"

# VM STONITH
pcs stonith create vm-fence fence_xvm \
    pcmk_host_map="node1:vm1;node2:vm2"

# Power switch STONITH
pcs stonith create power-fence fence_apc_snmp \
    ipaddr=192.168.1.201 community=public \
    pcmk_host_list="node1 node2"

# Verify STONITH
pcs stonith show
pcs stonith describe fence_ipmilan
```

### 2.7 Monitoring and Debugging

```bash
# Cluster monitoring
crm_mon                           # Interactive monitor
crm_mon -1                        # One-time display
crm_mon --as-xml                  # XML output
crm_mon --watch                   # Watch mode
crm_mon --resource-name vip       # Filter by resource

# Log analysis
journalctl -u pacemaker -f
tail -f /var/log/pacemaker/pacemaker.log

# Resource debugging
pcs resource show-status
pcs resource debug-start web-group
pcs resource debug-stop web-group

# Configuration verification
pcs config verify
pcs status --full

# Simulation
crm_simulate --simulate
crm_simulate --show-score
crm_simulate --dot-file=graph.dot
```

### 2.8 Troubleshooting

```bash
# Check cluster health
pcs status

# Check resource status
pcs status resources
pcs resource show

# Check node status
pcs status nodes
pcs node status

# Check constraints
pcs constraint show

# Check STONITH
pcs stonith show

# Restart cluster
pcs cluster stop --all
pcs cluster start --all

# Maintenance mode
pcs property set maintenance-mode=true
# Perform maintenance
pcs property set maintenance-mode=false
```

---

## 3. QEMU/KVM Cheatsheet

### 3.1 Installation and Setup

```bash
# Check hardware support
lscpu | grep Virtualization
egrep -c '(vmx|svm)' /proc/cpuinfo

# Installation
apt-get install qemu-kvm libvirt-daemon-system libvirt-clients virt-manager  # Debian/Ubuntu
yum install qemu-kvm libvirt virt-install virt-manager                    # RHEL/CentOS

# Enable and start services
systemctl enable libvirtd
systemctl start libvirtd

# Add user to libvirt group
usermod -aG libvirt $(whoami)

# Verify installation
virsh version
qemu-system-x86_64 --version
```

### 3.2 VM Management Commands

```bash
# List VMs
virsh list                        # Running VMs
virsh list --all                 # All VMs

# VM lifecycle
virsh start vm1                  # Start VM
virsh shutdown vm1               # Shutdown VM
virsh destroy vm1               # Force stop VM
virsh suspend vm1               # Suspend VM
virsh resume vm1                # Resume VM

# VM information
virsh dominfo vm1               # VM information
virsh dumpxml vm1              # VM XML configuration
virsh vcpuinfo vm1             # VCPU information
virsh dommemstat vm1           # Memory statistics

# VM configuration
virsh define vm1.xml           # Define VM from XML
virsh undefine vm1             # Remove VM definition
virsh autostart vm1            # Enable autostart
virsh autostart --disable vm1   # Disable autostart

# Resource management
virsh setvcpus vm1 4 --live   # Set VCPUS
virsh setmem vm1 4G --live     # Set memory
```

### 3.3 VM Creation

```bash
# Create VM from ISO
virt-install \
  --name vm1 \
  --memory 2048 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/vm1.qcow2,size=20 \
  --cdrom /path/to/iso.iso \
  --network network=default \
  --graphics spice \
  --os-variant ubuntu20.04

# Create VM from disk image
virt-install \
  --name vm2 \
  --memory 4096 \
  --vcpus 4 \
  --disk path=/var/lib/libvirt/images/vm2.qcow2 \
  --import \
  --os-variant rhel8

# Clone VM
virt-clone --original vm1 --name vm2 --file /var/lib/libvirt/images/vm2.qcow2
```

### 3.4 Direct QEMU Commands

```bash
# Basic VM
qemu-system-x86_64 \
  -name vm1 \
  -m 2048 \
  -smp 2 \
  -drive file=/path/to/disk.qcow2,format=qcow2

# Enable KVM acceleration
qemu-system-x86_64 -enable-kvm \
  -name vm1 -m 2048 -smp 2 \
  -drive file=disk.qcow2,format=qcow2

# Network configuration
qemu-system-x86_64 \
  -netdev user,id=net0,hostfwd=tcp::2222-:22 \
  -device virtio-net-pci,netdev=net0

# Headless mode
qemu-system-x86_64 -nographic \
  -name vm1 -m 2048 -smp 2 \
  -drive file=disk.qcow2,format=qcow2

# Attach ISO
qemu-system-x86_64 \
  -drive file=disk.qcow2,format=qcow2 \
  -cdrom /path/to/iso.iso \
  -boot d
```

### 3.5 Network Configuration

```bash
# List networks
virsh net-list
virsh net-list --all

# Create network
virsh net-define network.xml
virsh net-start network-name
virsh net-autostart network-name

# Network information
virsh net-info network-name
virsh net-dumpxml network-name

# Attach network to VM
virsh attach-interface vm1 network default
```

### 3.6 Storage Management

```bash
# Create disk image
qemu-img create -f qcow2 /path/to/disk.qcow2 20G

# Convert image formats
qemu-img convert -f raw -O qcow2 disk.raw disk.qcow2

# Resize disk
qemu-img resize /path/to/disk.qcow2 +10G

# Disk information
qemu-img info /path/to/disk.qcow2

# Check disk integrity
qemu-img check /path/to/disk.qcow2

# Create snapshot
qemu-img snapshot -c snapshot-name /path/to/disk.qcow2

# List snapshots
qemu-img snapshot -l /path/to/disk.qcow2

# Apply snapshot
qemu-img snapshot -a snapshot-name /path/to/disk.qcow2
```

### 3.7 Monitoring and Debugging

```bash
# VM performance
virsh dommemstat vm1
virsh domblklist vm1
virsh domiflist vm1

# Top-like monitoring
virt-top

# Network monitoring
virsh iface-list
virsh net-dhcp-leases network-name

# Console access
virsh console vm1              # Serial console
virsh attach-disk vm1 /path/to/disk vdb  # Attach disk

# QEMU monitor
virsh qemu-monitor-command vm1 --hmp
virsh qemu-monitor-command vm1 --cmd '{"execute":"query-status"}'
```

### 3.8 Live Migration

```bash
# Live migration
virsh migrate --live --persistent vm1 qemu+tcp://dest/system

# Migration with storage
virsh migrate --live --copy-storage-all vm1 \
    qemu+tcp://dest/system

# Migration without shared storage
virsh migrate --live --copy-storage-all \
    --persistent vm1 qemu+tcp://dest/system

# Check migration progress
watch virsh domjobinfo vm1
```

### 3.9 Device Passthrough

```bash
# Enable IOMMU
# Add to kernel command line: intel_iommu=on amd_iommu=on

# Load VFIO modules
modprobe vfio vfio_pci vfio_iommu_type1

# Find device
lspci -nn

# Bind device to VFIO
echo "0000:01:00.0" > /sys/bus/pci/drivers/vfio-pci/bind

# VM XML configuration
<hostdev mode='subsystem' type='pci' managed='yes'>
  <driver name='vfio'/>
  <source>
    <address domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
  </source>
</hostdev>
```

### 3.10 Troubleshooting

```bash
# Check virtualization support
lscpu | grep Virtualization

# Check libvirt status
systemctl status libvirtd

# Check VM status
virsh list --all

# VM logs
tail -f /var/log/libvirt/qemu/vm1.log
journalctl -u libvirtd -f

# Network issues
virsh net-list
virsh net-info default
brctl show

# Disk issues
qemu-img check /path/to/disk.qcow2
```

---

## 4. Quick Reference Tables

### 4.1 Corosync Transport Types

| Transport | Description | Pros | Cons |
|-----------|-------------|-------|-------|
| udp | IP multicast | Simple setup | Requires multicast support |
| udpu | Unicast UDP | No multicast needed | More configuration |
| knet | Kronosnet | Redundant paths | Newer, less tested |

### 4.2 Pacemaker Resource Classes

| Class | Description | Location |
|-------|-------------|----------|
| OCF | Open Cluster Framework | /usr/lib/ocf/resource.d/ |
| LSB | Linux Standard Base | /etc/init.d/ |
| systemd | Systemd service | /usr/lib/systemd/ |
| Upstart | Upstart job | /etc/init/ |

### 4.3 Pacemaker OCF Return Codes

| Code | Constant | Description |
|------|----------|-------------|
| 0 | OCF_SUCCESS | Operation successful |
| 1 | OCF_ERR_GENERIC | Generic error |
| 2 | OCF_ERR_ARGS | Invalid arguments |
| 3 | OCF_ERR_UNIMPLEMENTED | Not implemented |
| 4 | OCF_ERR_PERM | Permission denied |
| 5 | OCF_ERR_INSTALLED | Not installed |
| 6 | OCF_ERR_CONFIGURED | Misconfigured |
| 7 | OCF_NOT_RUNNING | Resource not running |

### 4.4 QEMU Disk Formats

| Format | Description | Use Case |
|--------|-------------|----------|
| qcow2 | QEMU Copy-On-Write | Most common, supports snapshots |
| raw | Raw disk image | Best performance |
| vmdk | VMware format | VMware compatibility |
| vdi | VirtualBox format | VirtualBox compatibility |

### 4.5 QEMU Network Models

| Model | Description | Performance |
|-------|-------------|--------------|
| virtio-net | Virtio network | Best |
| e1000 | Intel e1000 | Good |
| rtl8139 | Realtek RTL8139 | Moderate |

### 4.6 KVM CPU Modes

| Mode | Description | Performance |
|------|-------------|--------------|
| host-passthrough | Pass all CPU features | Best |
| host-model | Emulate host CPU | Good |
| custom | Specify CPU features | Variable |

---

## 5. Common Workflows

### 5.1 Corosync Cluster Setup

```bash
# 1. Install Corosync
apt-get install corosync

# 2. Generate authentication key
corosync-keygen
scp /etc/corosync/authkey node2:/etc/corosync/
scp /etc/corosync/authkey node3:/etc/corosync/

# 3. Configure corosync.conf
vim /etc/corosync/corosync.conf

# 4. Start Corosync
systemctl enable corosync
systemctl start corosync

# 5. Verify cluster
corosync-cfgtool -s
corosync-quorumtool -s
```

### 5.2 Pacemaker Cluster Setup

```bash
# 1. Install packages
apt-get install pacemaker pcs

# 2. Start and enable pcsd
systemctl enable pcsd
systemctl start pcsd

# 3. Authenticate nodes
pcs cluster auth node1 node2 node3 -u hacluster

# 4. Create cluster
pcs cluster setup --name mycluster node1 node2 node3

# 5. Start cluster
pcs cluster start --all
pcs cluster enable --all

# 6. Configure properties
pcs property set stonith-enabled=true
pcs property set no-quorum-policy=stop
```

### 5.3 Adding a Resource to Pacemaker

```bash
# 1. Create resource
pcs resource create vip ocf:heartbeat:IPaddr2 \
    ip=192.168.1.100 cidr_netmask=24 \
    op monitor interval=30s

# 2. Verify resource
pcs status resources
pcs resource show vip

# 3. Start resource
pcs resource start vip

# 4. Add ordering constraint
pcs constraint order start vip then apache

# 5. Add colocation constraint
pcs constraint colocation add apache with vip
```

### 5.4 Creating a QEMU VM

```bash
# 1. Create disk image
qemu-img create -f qcow2 /var/lib/libvirt/images/vm1.qcow2 20G

# 2. Create VM
virt-install \
  --name vm1 \
  --memory 2048 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/vm1.qcow2 \
  --cdrom /path/to/iso.iso \
  --network network=default \
  --graphics spice

# 3. Verify VM
virsh list --all
virsh dominfo vm1

# 4. Start VM
virsh start vm1

# 5. Access console
virsh console vm1
```

### 5.5 Live Migration Workflow

```bash
# Source host
1. Ensure VM is running
virsh list

2. Verify storage is shared or available
ls -lh /path/to/shared/storage

3. Start migration
virsh migrate --live --persistent vm1 qemu+tcp://dest/system

4. Monitor migration
watch virsh domjobinfo vm1

# Destination host
1. Verify VM arrived
virsh list --all

2. Verify VM is running
virsh dominfo vm1
```

---

## 6. Nifty Behaviors and Tips

### 6.1 Corosync Nifty Behaviors

#### Kronosnet Multi-Link Transport
```conf
totem {
    transport: knet
    interface {
        ringnumber: 0
        bindnetaddr: 192.168.1.10
    }
    interface {
        ringnumber: 1
        bindnetaddr: 192.168.2.10
    }
}
```
**Nifty**: Automatic failover between network links, better performance

#### Token Timeout Optimization
```conf
totem {
    token: 5000           # Reduce for faster failure detection
    token_retransmit: 250
}
```
**Nifty**: Faster cluster response to failures, but may increase false positives

#### Configuration Database Queries
```bash
# Query all configuration
corosync-cmapctl

# Query specific subtree
corosync-cmapctl -t nodelist

# Query specific key
corosync-cmapctl runtime.totem.token
```
**Nifty**: Runtime inspection of cluster state without configuration reload

### 6.2 Pacemaker Nifty Behaviors

#### Resource Stickiness with Score
```xml
<meta_attributes>
    <nvpair name="resource-stickiness" value="100"/>
    <nvpair name="migration-threshold" value="3"/>
</meta_attributes>
```
**Nifty**: Resources prefer current location but move after failures

#### Batch Constraint Updates
```bash
pcs constraint order start vip then apache
pcs constraint colocation add apache with vip
pcs constraint location web-group prefers node1=100
```
**Nifty**: Constraints are applied as a set, atomic configuration

#### Resource Clone with Notification
```xml
<clone id="notify-clone">
    <primitive class="ocf" provider="heartbeat" type="apache"/>
    <meta_attributes>
        <nvpair name="notify" value="true"/>
    </meta_attributes>
</clone>
```
**Nifty**: Resource receives notifications when clone state changes

#### Symmetric Cluster Mode
```bash
pcs property set symmetric-cluster=false
pcs resource create web-server apache \
    meta allowed-nodes="node1 node2"
```
**Nifty**: Control where resources can run, prevent unwanted placement

### 6.3 QEMU/KVM Nifty Behaviors

#### QMP (QEMU Monitor Protocol)
```bash
# Connect to QMP
echo '{"execute":"qmp_capabilities"}' | socat - UNIX-CONNECT:/var/run/libvirt/qemu/vm1.monitor

# Query VM status
echo '{"execute":"query-status"}' | socat - UNIX-CONNECT:/var/run/libvirt/qemu/vm1.monitor
```
**Nifty**: Programmatic control of running VMs

#### Virtio Multiqueue
```xml
<interface type='network'>
  <model type='virtio'/>
  <driver name='vhost' queues='4'/>
</interface>
```
**Nifty**: Multi-queue network, better throughput and parallelism

#### Memory Ballooning
```bash
# Attach balloon device
virsh attach-device vm1 balloon.xml

# Adjust memory
virsh setmem vm1 2G
```
**Nifty**: Dynamically adjust VM memory without reboot

#### vCPU Hotplug
```bash
# Add vCPUs online
virsh setvcpus vm1 4 --live

# Pin vCPUs
virsh vcpupin vm1 0-3 0-3
```
**Nifty**: Dynamically adjust CPU count, improved performance

#### Block Dirty Bitmaps
```bash
# Enable dirty bitmaps
qemu-img bitmap --add /path/to/disk.qcow2 bitmap-name

# Create incremental backup using bitmaps
qemu-img create -F qcow2 -f qcow2 -b /path/to/disk.qcow2 \
    -B bitmap-name /path/to/incremental.qcow2
```
**Nifty**: Efficient incremental backups, track changed blocks

#### QEMU Guest Agent
```bash
# Install agent in VM
apt-get install qemu-guest-agent

# Execute commands from host
virsh qemu-agent-command vm1 '{"execute":"guest-ping"}'
```
**Nifty**: Execute commands inside VM without network access

### 6.4 General Nifty Behaviors

#### libvirt Storage Pools
```bash
# Define storage pool
virsh pool-define-as mypool dir /path/to/pool

# Start and autostart
virsh pool-start mypool
virsh pool-autostart mypool

# Create volume in pool
virsh vol-create-as mypool vm1.qcow2 20G --format qcow2
```
**Nifty**: Centralized storage management, automated volume creation

#### libvirt Networks with NAT
```bash
# Create NAT network
virsh net-create <(cat <<EOF
<network>
  <name>mynetwork</name>
  <forward mode='nat'/>
  <ip address='192.168.100.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.100.2' end='192.168.100.254'/>
    </dhcp>
  </ip>
</network>
EOF
)
```
**Nifty**: Automatic DHCP, NAT routing, isolated VM network

#### Pacemaker Resource Agents with Metadata
```bash
# Display resource metadata
crm_resource --resource vip --meta
# Or
ocf_ipaddr2 meta-data
```
**Nifty**: Understand resource capabilities, required parameters

#### Corosync Configuration Validation
```bash
# Validate configuration
corosync-cfgtool -k
```
**Nifty**: Test configuration before applying to production

---

## 7. Performance Optimization Tips

### 7.1 Corosync Optimization

```bash
# Reduce token timeout for faster detection
totem {
    token: 3000
    token_retransmit: 150
}

# Increase network buffer size
totem {
    window_size: 100
    max_messages: 50
}

# Use Kronosnet for better performance
totem {
    transport: knet
}
```

### 7.2 Pacemaker Optimization

```bash
# Reduce monitor interval for faster failure detection
pcs resource update vip \
    op monitor interval=10s

# Adjust migration threshold
pcs resource update apache \
    meta migration-threshold=1 failure-timeout=60s

# Use batched actions
pcs resource op create apache monitor interval=30s timeout=20s
```

### 7.3 QEMU/KVM Optimization

```bash
# Use host CPU passthrough
cpu mode='host-passthrough'

# Enable hugepages
<memoryBacking>
  <hugepages>
    <page size='2048' unit='KiB'/>
  </hugepages>
</memoryBacking>

# Use virtio drivers
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2'/>
  <target dev='vda' bus='virtio'/>
</disk>

# Enable IO threads
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2' io='threads'/>
</disk>

# Use multiple queues for network
<interface type='network'>
  <model type='virtio'/>
  <driver name='vhost' queues='4'/>
</interface>

# CPU pinning
<vcpu placement='static'>4</vcpu>
<cputune>
  <vcpupin vcpu='0' cpuset='0'/>
  <vcpupin vcpu='1' cpuset='1'/>
  <vcpupin vcpu='2' cpuset='2'/>
  <vcpupin vcpu='3' cpuset='3'/>
</cputune>
```

---

## 8. Security Considerations

### 8.1 Corosync Security

```bash
# Protect authentication key
chmod 600 /etc/corosync/authkey

# Use encrypted transport (knet)
totem {
    transport: knet
    knet {
        crypto_cipher: aes256
        crypto_hash: sha256
    }
}

# Isolate cluster network
# Use dedicated VLAN or physical network
```

### 8.2 Pacemaker Security

```bash
# Always enable STONITH
pcs property set stonith-enabled=true

# Secure cluster communication
# Use corosync with authentication

# Limit cluster daemon permissions
chmod 644 /etc/corosync/corosync.conf

# Use SELinux/AppArmor profiles
# Enforce mandatory access control
```

### 8.3 QEMU/KVM Security

```bash
# Use AppArmor/SELinux profiles
semanage fcontext -a -t virt_image_t "/path/to/images(/.*)?"
restorecon -R /path/to/images

# Use virtio drivers for better isolation
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2'/>
  <target dev='vda' bus='virtio'/>
</disk>

# Enable seccomp filters
<seccomp>
  <filter action='drop'>
    <rule action='errno'>
      <syscall name='getuid'/>
    </rule>
  </filter>
</seccomp>

# Use IOMMU for device passthrough
# Add to kernel command line: intel_iommu=on amd_iommu=on
```

---

## 9. Reference Links

### Corosync
- Official Site: https://corosync.github.io/corosync/
- Source Code: https://github.com/corosync/corosync
- Mailing List: corosync@oss.redhat.com

### Pacemaker
- Official Site: https://www.clusterlabs.org/pacemaker/
- Source Code: https://github.com/ClusterLabs/pacemaker
- Documentation: https://www.clusterlabs.org/pacemaker/doc/

### QEMU
- Official Site: https://www.qemu.org/
- Source Code: https://gitlab.com/qemu-project/qemu
- Documentation: https://www.qemu.org/documentation/

### KVM
- Official Site: https://www.linux-kvm.org/
- Source Code: https://github.com/torvalds/linux/tree/master/virt/kvm
- Wiki: https://www.linux-kvm.org/page/Documents

### libvirt
- Official Site: https://libvirt.org/
- Source Code: https://gitlab.com/libvirt/libvirt
- Documentation: https://libvirt.org/docs/

---

## 10. Common Error Messages and Solutions

### Corosync Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Unable to access authkey" | Wrong permissions | chmod 600 /etc/corosync/authkey |
| "Cluster not configured" | Missing corosync.conf | Create configuration file |
| "No quorum" | Node loss | Verify network, check node count |

### Pacemaker Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "STONITH not configured" | No fencing device | Configure STONITH |
| "Resource failed to start" | Configuration error | Check resource parameters |
| "Constraint violation" | Invalid constraint | Review constraints |

### QEMU/KVM Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Could not access KVM" | No virtualization support | Enable VT-x/AMD-V |
| "Permission denied" | Wrong file permissions | chmod 644 disk image |
| "No bootable device" | Missing boot device | Attach disk or ISO |

---

This cheatsheet provides quick reference commands, configurations, and tips for Corosync, Pacemaker, and QEMU/KVM. Use it as a handy reference for daily operations and troubleshooting.
