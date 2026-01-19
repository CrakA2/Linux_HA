# libvirt

Virtualization API and daemon for managing virtualization capabilities.

## Architecture

```mermaid
graph TB
    A[libvirtd Daemon] --> B[QEMU/KVM]
    A --> C[LXC Containers]
    A --> D[Xen]
    
    E[Management Tools] --> F[virsh CLI]
    E --> G[virt-manager GUI]
    E --> H[API Bindings]
    
    F --> A
    G --> A
    H --> A
```

## Key Features

- Unified API for multiple hypervisors
- XML-based VM configuration
- Network and storage management
- Secure API with authentication
- Live migration support

## Quick Commands

```bash
# List VMs
virsh list --all

# VM operations
virsh start vm1
virsh shutdown vm1
virsh dumpxml vm1

# Network management
virsh net-list
virsh net-define network.xml
```

## Source Code

- Repository: https://gitlab.com/libvirt/libvirt
- Documentation: https://libvirt.org/docs/
