# Linux High Availability and Virtualization

Comprehensive course materials and reference documentation for Linux High Availability (HA) clustering and virtualization technologies.

## Documentation

The complete documentation is hosted on [GitHub Pages](https://YOUR_USERNAME.github.io/Linux_HA/) and includes:

- **Cluster Technologies**: Corosync, Pacemaker
- **Virtualization**: QEMU, KVM, libvirt, virsh, Open vSwitch
- **Storage**: CEPH, iSCSI, NVMe-oF, GFS2
- **Networking**: libvirt networking configurations

## Quick Start

```bash
# Clone repository
git clone https://github.com/YOUR_USERNAME/Linux_HA.git
cd Linux_HA

# View documentation locally
pip install -r requirements.txt
mkdocs serve
# Open http://127.0.0.1:8000
```

## Topics Covered

### Cluster Management
- Corosync messaging and quorum
- Pacemaker resource management
- STONITH and high availability configuration

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

## Development

To build the documentation site:

```bash
pip install -r requirements.txt
mkdocs build
```

## License

Course materials follow documentation licenses of their respective projects:
- Corosync: BSD License
- Pacemaker: GPL-2.0 and LGPL-2.1
- QEMU/KVM: GPL-2.0
- CEPH: GPL-2.0, LGPL-2.1, LGPL-3.0
- GFS2: GPL-2.0
- libvirt: LGPL-2.1
- Open vSwitch: Apache 2.0

## Resources

- [Live Documentation](https://YOUR_USERNAME.github.io/Linux_HA/)
- [CEPH Documentation](https://docs.ceph.com/)
- [ClusterLabs](https://www.clusterlabs.org/)
- [KVM Documentation](https://www.linux-kvm.org/)
