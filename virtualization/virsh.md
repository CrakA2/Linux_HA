# virsh

Command-line interface for managing libvirt virtualization with comprehensive VM lifecycle management.

## Overview

virsh provides a unified interface for managing virtual machines, networks, storage, and other libvirt resources.

**virsh Connection Modes**:
```bash
# Local connection
virsh

# Remote connection
virsh -c qemu+ssh://user@host/system
virsh -c qemu+tcp://host/system
virsh -c qemu+tls://host/system

# Read-only connection
virsh -r
```

## Domain Lifecycle Management

### VM Creation

```bash
# Create VM from XML
virsh define vm1.xml

# Create VM from ISO (interactive)
virt-install --name vm1 --memory 2048 --vcpus 2 \
    --disk path=/var/lib/libvirt/images/vm1.qcow2,size=20 \
    --cdrom /path/to/ubuntu.iso \
    --graphics vnc

# Clone existing VM
virt-clone --original vm1 --name vm2 --file /var/lib/libvirt/images/vm2.qcow2
```

### VM Start/Stop

```bash
# Start VM
virsh start vm1

# Start VM with autostart
virsh start vm1 --console

# Shutdown VM (graceful)
virsh shutdown vm1

# Shutdown with force
virsh shutdown vm1 --mode=force

# Destroy VM (force stop)
virsh destroy vm1

# Reboot VM
virsh reboot vm1

# Reset VM (hardware reset)
virsh reset vm1

# Suspend VM
virsh suspend vm1

# Resume VM
virsh resume vm1
```

### VM Deletion

```bash
# Undefine VM (remove configuration)
virsh undefine vm1

# Undefine with storage
virsh undefine vm1 --remove-all-storage

# Undefine with snapshots
virsh undefine vm1 --remove-all-storage --snapshots-metadata

# Undefine with NVRAM
virsh undefine vm1 --nvram
```

### VM Autostart

```bash
# Enable autostart
virsh autostart vm1

# Disable autostart
virsh autostart --disable vm1

# List autostart VMs
virsh list --autostart
```

## VM Information and Monitoring

### Domain Information

```bash
# Domain information
virsh dominfo vm1

# Domain ID and UUID
virsh domid vm1
virsh domuuid vm1

# Domain state
virsh domstate vm1

# Domain flags
virsh domcontrol vm1

# Domain time
virsh domtime vm1
```

### CPU Information

```bash
# VCPU information
virsh vcpuinfo vm1

# VCPU pinning
virsh vcpupin vm1

# Set VCPU pinning
virsh vcpupin vm1 0 0-1
virsh vcpupin vm1 1 2-3

# VCPU count
virsh vcpuinfo vm1
virsh setvcpus vm1 4 --live
virsh setvcpus vm1 4 --config
virsh setvcpus vm1 4 --current

# Maximum VCPU
virsh maxvcpus vm1
```

### Memory Information

```bash
# Memory statistics
virsh dommemstat vm1

# Set memory
virsh setmem vm1 2G --live
virsh setmem vm1 2G --config
virsh setmem vm1 2G --current

# Maximum memory
virsh setmaxmem vm1 4G --live

# Memory balloon
virsh dommemstat vm1 | grep actual
```

### Device Information

```bash
# Block devices
virsh domblklist vm1
virsh domblkinfo vm1 vda

# Network devices
virsh domiflist vm1
virsh domifstat vm1 vnet0

# All devices
virsh dumpxml vm1 | grep '<disk'
virsh dumpxml vm1 | grep '<interface'
```

### Performance Statistics

```bash
# Block statistics
virsh domblkstat vm1

# Network statistics
virsh domifstat vm1 vnet0

# CPU time
virsh cpu-stats vm1

# CPU usage (total)
virsh cpu-stats vm1 --total

# CPU usage (per vcpu)
virsh cpu-stats vm1
```

## VM Configuration

### XML Configuration

```bash
# Dump XML configuration
virsh dumpxml vm1

# Dump XML to file
virsh dumpxml vm1 > vm1.xml

# Edit XML configuration
virsh edit vm1

# Edit specific network interface
virsh net-edit default

# Validate XML
virsh define vm1.xml
```

### Device Management

```bash
# Attach device
virsh attach-device vm1 disk.xml

# Attach device from command
virsh attach-device vm1 --file disk.xml --persistent

# Detach device
virsh detach-device vm1 disk.xml

# Update device
virsh update-device vm1 disk.xml

# List devices
virsh domblklist vm1
virsh domiflist vm1
```

**Disk Device XML**:
```xml
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2'/>
  <source file='/var/lib/libvirt/images/disk.qcow2'/>
  <target dev='vdb' bus='virtio'/>
</disk>
```

**Network Device XML**:
```xml
<interface type='network'>
  <mac address='52:54:00:71:b1:b6'/>
  <source network='default'/>
  <model type='virtio'/>
</interface>
```

## Console Access

### Serial Console

```bash
# Connect to serial console
virsh console vm1

# Connect to specific console
virsh console vm1 --devname hvc0

# Force console connection
virsh console vm1 --force
```

### VNC Console

```bash
# Find VNC display
virsh vncdisplay vm1

# Find VNC port
virsh qemu-monitor-command vm1 "info vnc"
```

### QEMU Monitor

```bash
# QEMU monitor command
virsh qemu-monitor-command vm1 "info status"
virsh qemu-monitor-command vm1 "info cpus"
virsh qemu-monitor-command vm1 "info block"
virsh qemu-monitor-command vm1 "info network"

# QMP command
virsh qemu-monitor-command vm1 '{"execute":"query-status"}'
```

## Snapshot Management

### Creating Snapshots

```bash
# Create snapshot
virsh snapshot-create vm1

# Create snapshot with name
virsh snapshot-create-as vm1 snap1

# Create snapshot from XML
virsh snapshot-create vm1 snap.xml

# Create snapshot with description
virsh snapshot-create-as vm1 snap1 \
    --description "Snapshot before update"
```

**Snapshot XML**:
```xml
<domainsnapshot>
  <name>snap1</name>
  <description>Snapshot before update</description>
  <memory snapshot='no'/>
  <disks>
    <disk name='vda' snapshot='internal'/>
  </disks>
</domainsnapshot>
```

### Managing Snapshots

```bash
# List snapshots
virsh snapshot-list vm1

# List snapshots with tree
virsh snapshot-list vm1 --tree

# Snapshot information
virsh snapshot-info vm1 snap1

# Dump snapshot XML
virsh snapshot-dumpxml vm1 snap1

# Snapshot parent
virsh snapshot-parent vm1 snap1
```

### Reverting Snapshots

```bash
# Revert to snapshot
virsh snapshot-revert vm1 snap1

# Revert with force
virsh snapshot-revert vm1 snap1 --force

# Revert current state
virsh snapshot-revert vm1 --current
```

### Deleting Snapshots

```bash
# Delete snapshot
virsh snapshot-delete vm1 snap1

# Delete all snapshots
virsh snapshot-list vm1 --name | xargs -I {} virsh snapshot-delete vm1 {}
```

## Migration

### Live Migration

```bash
# Live migration
virsh migrate --live vm1 qemu+ssh://dest/system

# Live migration with compression
virsh migrate --live --compress vm1 qemu+ssh://dest/system

# Live migration with timeout
virsh migrate --live --timeout 600 vm1 qemu+ssh://dest/system

# Live migration with bandwidth limit
virsh migrate --live --bandwidth 100 vm1 qemu+ssh://dest/system
```

### Cold Migration

```bash
# Cold migration
virsh migrate vm1 qemu+ssh://dest/system

# Cold migration with XML
virsh migrate --xml vm1.xml qemu+ssh://dest/system
```

### Migration Options

| Option | Description |
|--------|-------------|
| --live | Live migration |
| --p2p | Peer-to-peer migration |
| --direct | Direct migration |
| --tunnelled | Tunnelled migration |
| --persistent | Persist on destination |
| --undefinesource | Undefine source |
| --persistent | Make destination persistent |
| --suspend | Suspend on migration |
| --copy-storage-all | Copy all storage |
| --copy-storage-inc | Copy storage incrementally |
| --compress | Compress migration data |
| --abort-on-error | Abort on errors |
| --timeout | Set timeout |

### Migration Monitoring

```bash
# Monitor migration
virsh migrate --live --verbose vm1 qemu+ssh://dest/system

# Monitor QEMU monitor
virsh qemu-monitor-command vm1 "info migrate"

# Cancel migration
virsh migrate vm1 --abort
```

## Network Management

### Network Operations

```bash
# List networks
virsh net-list
virsh net-list --all
virsh net-list --inactive
virsh net-list --active

# Network information
virsh net-info default

# Dump network XML
virsh net-dumpxml default

# Define network
virsh net-define network.xml

# Undefine network
virsh net-undefine default

# Start network
virsh net-start default

# Stop network
virsh net-stop default

# Network autostart
virsh net-autostart default
virsh net-autostart --disable default
```

**Network XML**:
```xml
<network>
  <name>default</name>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```

## Storage Management

### Storage Pool Operations

```bash
# List storage pools
virsh pool-list
virsh pool-list --all
virsh pool-list --inactive
virsh pool-list --active

# Pool information
virsh pool-info default

# Dump pool XML
virsh pool-dumpxml default

# Define pool
virsh pool-define pool.xml

# Undefine pool
virsh pool-undefine default

# Build pool
virsh pool-build default

# Start pool
virsh pool-start default

# Stop pool
virsh pool-stop default

# Pool autostart
virsh pool-autostart default
virsh pool-autostart --disable default
```

**Storage Pool XML**:
```xml
<pool type='dir'>
  <name>default</name>
  <target>
    <path>/var/lib/libvirt/images</path>
    <permissions>
      <mode>0755</mode>
      <owner>-1</owner>
      <group>-1</group>
    </permissions>
  </target>
</pool>
```

### Storage Volume Operations

```bash
# List volumes
virsh vol-list default

# Volume information
virsh vol-info vm1.qcow2

# Dump volume XML
virsh vol-dumpxml vm1.qcow2

# Create volume
virsh vol-create-as default vm1.qcow2 20G

# Create volume from XML
virsh vol-create default volume.xml

# Delete volume
virsh vol-delete default vm1.qcow2

# Upload to volume
virsh vol-upload default vm1.qcow2 disk.qcow2

# Download from volume
virsh vol-download default vm1.qcow2 disk.qcow2

# Clone volume
virsh vol-clone default vm1.qcow2 vm2.qcow2

# Wipe volume
virsh vol-wipe default vm1.qcow2

# Resize volume
virsh vol-resize default vm1.qcow2 30G
```

## Secret Management

### Secret Operations

```bash
# Define secret
cat > secret.xml <<EOF
<secret ephemeral='no' private='no'>
  <description>libvirt secret</description>
  <usage type='ceph'>
    <name>client.admin secret</name>
  </usage>
</secret>
EOF

virsh secret-define secret.xml

# Set secret value
virsh secret-set-value secret.xml /path/to/key

# Get secret value
virsh secret-get-value secret.xml

# Delete secret
virsh secret-undefine secret.xml

# List secrets
virsh secret-list
```

## Advanced Commands

### Guest Operations

```bash
# Guest agent command
virsh qemu-agent-command vm1 '{"execute":"guest-ping"}'

# Guest shutdown
virsh qemu-agent-command vm1 '{"execute":"guest-shutdown","arguments":{"mode":"poweroff"}}'

# Guest info
virsh qemu-agent-command vm1 '{"execute":"guest-info"}'
```

### Host Management

```bash
# Node info
virsh nodeinfo

# Host capabilities
virsh capabilities

# CPU models
virsh cpu-models

# Hostname
virsh hostname

# Sysinfo
virsh sysinfo
```

### Version Information

```bash
# Version
virsh version

# Compiled information
virsh version

# Daemon version
virsh version --daemon
```

## Troubleshooting

### VM Issues

```bash
# Check VM logs
tail -f /var/log/libvirt/qemu/vm1.log

# Check libvirt logs
tail -f /var/log/libvirt/libvirtd.log

# Check VM status
virsh domstate vm1
virsh domcontrol vm1

# Check XML configuration
virsh dumpxml vm1

# Check console
virsh console vm1
```

### Network Issues

```bash
# Check network status
virsh net-list --all

# Check network XML
virsh net-dumpxml default

# Check network logs
journalctl -u libvirtd -f

# Check bridge
brctl show
ip addr show virbr0
```

### Storage Issues

```bash
# Check pool status
virsh pool-list --all

# Check pool XML
virsh pool-dumpxml default

# Check volume status
virsh vol-list default

# Check storage logs
journalctl -u libvirtd -f
```

## Best Practices

1. **Use --persistent** for permanent changes
2. **Create snapshots** before major changes
3. **Test migration** before production
4. **Use proper storage** for volumes
5. **Monitor VMs** regularly
6. **Use autostart** for critical VMs
7. **Document configurations** with XML
8. **Test backups** regularly
9. **Use virsh edit** for configuration changes
10. **Check logs** for troubleshooting

## Quick Reference

### Common Commands

| Command | Description |
|---------|-------------|
| `virsh list` | List running VMs |
| `virsh list --all` | List all VMs |
| `virsh start vm1` | Start VM |
| `virsh shutdown vm1` | Shutdown VM |
| `virsh destroy vm1` | Force stop VM |
| `virsh reboot vm1` | Reboot VM |
| `virsh suspend vm1` | Suspend VM |
| `virsh resume vm1` | Resume VM |
| `virsh define vm1.xml` | Define VM |
| `virsh undefine vm1` | Remove VM |
| `virsh dumpxml vm1` | Export XML |
| `virsh edit vm1` | Edit XML |
| `virsh dominfo vm1` | VM information |
| `virsh domstate vm1` | VM state |
| `virsh console vm1` | Connect console |
| `virsh snapshot-create vm1` | Create snapshot |
| `virsh snapshot-list vm1` | List snapshots |
| `virsh snapshot-revert vm1 snap1` | Revert snapshot |
| `virsh migrate --live vm1 dest` | Live migration |

## Documentation

- **virsh Manual**: `man virsh`
- **libvirt Documentation**: https://libvirt.org/docs/
- **virsh Cheat Sheet**: https://libvirt.org/virshcmdref/