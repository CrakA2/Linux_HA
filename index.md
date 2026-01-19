# Linux High Availability and Virtualization

Comprehensive course materials for Linux High Availability (HA) clustering and virtualization technologies including Corosync, Pacemaker, QEMU, KVM, CEPH, iSCSI, FC/NVMe-oF, GFS2, libvirt, virsh, and Open vSwitch.

## Table of Contents

### Core Technologies

- [Cluster Technologies](cluster/index.md) - Corosync, Pacemaker
- [Virtualization](virtualization/index.md) - QEMU, KVM, libvirt, virsh, Open vSwitch
- [Storage and File Systems](storage/index.md) - CEPH, iSCSI, NVMe-oF, GFS2

### Reference

- [Quick Reference](reference.md) - Command cheatsheets
- [Deployment Workflows](deployment.md) - Setup guides
- [Troubleshooting](troubleshooting-code-behavior.md) - Code behavior diagnostics

### Additional

- [Network and Switching](network/libvirt-network.md) - libvirt networking
- [Resources](resources.md) - External links
- [License](license.md) - License information

---

## Quick Start

```bash
# Clone repository
git clone https://github.com/CrakA2/Linux_HA.git
cd Linux_HA

# View locally
mkdocs serve
# Open http://127.0.0.1:8000
```

---

## Topics Covered

### Cluster Management

- Corosync messaging and quorum
- Pacemaker resource management
- STONITH and high availability configuration
- Cluster deployment and configuration

### Virtualization

- QEMU/KVM hypervisor setup
- libvirt VM management
- Open vSwitch networking
- Live migration and resource optimization

### Distributed Storage

- CEPH HCI deployment
- RBD block storage
- CephFS shared filesystem
- iSCSI/NVMe-oF gateways

### File Systems

- GFS2 clustered filesystem
- DLM lock management
- CLVM volume management

---

## Documentation Structure

```
docs/
├── cluster/           # Corosync, Pacemaker
├── virtualization/    # QEMU, KVM, libvirt, virsh, OVS
├── storage/          # CEPH, iSCSI, NVMe-oF, GFS2
├── network/          # libvirt networking
├── reference.md      # Quick command reference
├── deployment.md     # Deployment guides
└── troubleshooting-code-behavior.md  # Diagnostics
```

---

## Source Code References

| Technology | Repository | Documentation |
|-----------|---------------|-------------|
| Corosync | [corosync/corosync](https://github.com/corosync/corosync) | [corosync.github.io](https://corosync.github.io/corosync/) |
| Pacemaker | [ClusterLabs/pacemaker](https://github.com/ClusterLabs/pacemaker) | [clusterlabs.org](https://www.clusterlabs.org/pacemaker/doc/) |
| QEMU | [qemu-project/qemu](https://gitlab.com/qemu-project/qemu) | [qemu.org](https://www.qemu.org/documentation/) |
| KVM | [torvalds/linux](https://github.com/torvalds/linux/tree/master/virt/kvm) | [linux-kvm.org](https://www.linux-kvm.org/) |
| libvirt | [libvirt/libvirt](https://gitlab.com/libvirt/libvirt) | [libvirt.org](https://libvirt.org/docs/) |
| CEPH | [ceph/ceph](https://github.com/ceph/ceph) | [docs.ceph.com](https://docs.ceph.com/) |
| Open vSwitch | [openvswitch/ovs](https://github.com/openvswitch/ovs) | [openvswitch.org](https://docs.openvswitch.org/) |
| GFS2 | [torvalds/linux](https://github.com/torvalds/linux/tree/master/fs/gfs2) | [gfs2-utils](https://github.com/ceph/gfs2/) |

---

## Contributing

This is a personal knowledge base for Linux HA and virtualization technologies. Corrections and additions are welcome via pull requests to the GitHub repository.
