# Linux High Availability and Virtualization Course Materials

This repository contains comprehensive course materials for Linux High Availability (HA) and virtualization technologies. Each section includes detailed architecture documentation, deployment recommendations, code behavior analysis, and source code references.

## Repository Structure

```
Linux_HA/
├── README.md                    # This file
├── cheatsheets.md             # Comprehensive command reference
├── corosync/                 # Corosync Cluster Engine
├── pacemaker/                 # Pacemaker Cluster Resource Manager
├── qemu-kvm/                  # QEMU and KVM Virtualization
├── ceph-hci/                  # CEPH Hyper-Converged Infrastructure
├── iscsi-ceph/                # iSCSI with CEPH
├── fc-nvmeof-ceph/            # FC (NVMe-oF) with CEPH
└── gfs2/                      # GFS2 Shared Filesystem
```

## Course Topics

### 1. Corosync Cluster Engine
Cluster messaging and synchronization framework that provides virtual synchrony guarantees and quorum management.

- **Architecture**: TOTEM protocol, quorum system, configuration database
- **Deployment**: Multi-node cluster setup with Corosync
- **Integration**: Messaging layer for Pacemaker
- **Source Code**: https://github.com/corosync/corosync

[View Corosync Documentation](corosync/)

### 2. Pacemaker Cluster Resource Manager
Advanced cluster resource manager that coordinates configuration, start-up, monitoring, and recovery of interrelated services.

- **Architecture**: CRM, Policy Engine, Transition Engine, LRM, CIB
- **Resource Classes**: OCF, LSB, systemd, Upstart
- **Constraints**: Ordering, colocation, location constraints
- **STONITH**: Fencing integration for split-brain prevention
- **Source Code**: https://github.com/ClusterLabs/pacemaker

[View Pacemaker Documentation](pacemaker/)

### 3. QEMU and KVM
Full-system emulator and hardware-assisted virtualization for Linux.

- **QEMU**: Complete machine emulation with TCG dynamic translation
- **KVM**: Kernel-based virtual machine with hardware acceleration
- **Integration**: QEMU/KVM combined architecture
- **Source Codes**: 
  - QEMU: https://gitlab.com/qemu-project/qemu
  - KVM: https://github.com/torvalds/linux/tree/master/virt/kvm

[View QEMU/KVM Documentation](qemu-kvm/)

### 4. CEPH Hyper-Converged Infrastructure (HCI)
Distributed storage system providing object, block, and file storage.

- **Architecture**: MON, OSD, MGR, MDS, RGW components
- **Deployment**: Cephadm-based containerized deployment
- **CRUSH Algorithm**: Data placement and distribution
- **Source Code**: https://github.com/ceph/ceph

[View CEPH HCI Documentation](ceph-hci/)

### 5. iSCSI with CEPH
iSCSI gateway implementation for exporting CEPH RBD images over TCP/IP network.

- **Architecture**: LIO target framework, TCMU userspace backend
- **Integration**: RBD backend connecting to CEPH cluster
- **HA Support**: Multiple gateway configuration
- **Source Code**: https://github.com/ceph/ceph

[View iSCSI Documentation](iscsi-ceph/)

### 6. FC (NVMe-oF) with CEPH
NVMe over Fabrics gateway providing high-performance block access to CEPH.

- **Architecture**: NVMe/TCP target with RBD backend
- **SPDK Integration**: High-performance I/O path
- **HA Support**: Gateway groups with load balancing
- **RDMA Support**: High-performance network transport
- **Source Code**: https://github.com/ceph/ceph

[View FC/NVMe-oF Documentation](fc-nvmeof-ceph/)

### 7. GFS2
Global File System 2 for shared-disk file system in Linux clusters.

- **Architecture**: GFS2 kernel module, DLM distributed lock manager, CLVM
- **Integration**: Pacemaker/Corosync for cluster management
- **Features**: Quota, snapshots, journaling
- **Source Code**: https://github.com/torvalds/linux/tree/master/fs/gfs2

[View GFS2 Documentation](gfs2/)

## Cheatsheets

Comprehensive command reference for all topics including:

- Quick commands for Corosync
- Pacemaker resource and constraint management
- QEMU/KVM virtualization commands
- CEPH cluster operations
- iSCSI/NVMe-oF gateway configuration
- GFS2 filesystem management

[View Cheatsheets](cheatsheets.md)

## Quick Start

### Corosync
```bash
# Install
apt-get install corosync

# Bootstrap
corosync-keygen
systemctl start corosync
```

### Pacemaker
```bash
# Install
apt-get install pacemaker pcs

# Configure cluster
pcs cluster setup --name mycluster node1 node2 node3
pcs cluster start --all
```

### QEMU/KVM
```bash
# Install
apt-get install qemu-kvm libvirt

# Start VM
virsh start vm1
```

### CEPH
```bash
# Bootstrap
cephadm bootstrap --mon-ip <mon-ip>

# Add storage
ceph orch apply osd --all-available-devices
```

### iSCSI
```bash
# Create RBD image
rbd create rbd/disk1 --size 100G

# Deploy gateway
ceph orch apply iscsi gateway.yml
```

### NVMe-oF
```bash
# Create RBD image
rbd create rbd/disk1 --size 100G

# Deploy gateway
ceph orch apply nvmeof gateway.yml
```

### GFS2
```bash
# Create filesystem
mkfs.gfs2 -p lock_dlm -t mycluster -j 2 /dev/drbd/by-res/resource-data

# Mount
mount -t gfs2 -o noatime,nodiratime /dev/drbd/by-res/resource-data /mnt/gfs2
```

## Documentation Format

All course materials follow a consistent format:

- **Markdown** for easy reading
- **Mermaid diagrams** for architecture visualization
- **Scientific/technical language** for professional presentation
- **No emojis** for clean documentation
- **Source code links** for reference

## Reference Links

| Topic | Official Site | Source Code |
|--------|---------------|-------------|
| Corosync | https://corosync.github.io/corosync/ | https://github.com/corosync/corosync |
| Pacemaker | https://www.clusterlabs.org/pacemaker/ | https://github.com/ClusterLabs/pacemaker |
| QEMU | https://www.qemu.org/ | https://gitlab.com/qemu-project/qemu |
| KVM | https://www.linux-kvm.org/ | https://github.com/torvalds/linux/tree/master/virt/kvm |
| CEPH | https://docs.ceph.com/ | https://github.com/ceph/ceph |
| iSCSI | https://linux-iscsi.org/ | https://github.com/ceph/ceph |
| NVMe-oF | https://nvmexpress.org/ | https://github.com/ceph/ceph |
| GFS2 | /usr/share/doc/gfs2-utils/ | https://github.com/torvalds/linux/tree/master/fs/gfs2 |
| libvirt | https://libvirt.org/ | https://gitlab.com/libvirt/libvirt |

## Contribution

These course materials are designed for educational purposes. For corrections or additions, please refer to the official documentation of each respective project.

## License

Course materials follow the documentation licenses of their respective projects:
- Corosync: BSD License
- Pacemaker: GPL-2.0 and LGPL-2.1
- QEMU/KVM: GPL-2.0
- CEPH: GPL-2.0, LGPL-2.1, LGPL-3.0
- GFS2: GPL-2.0
