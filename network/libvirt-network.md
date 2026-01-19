# libvirt Networking

Network management for virtualization environments.

## Network Types

```mermaid
graph TB
    subgraph "libvirt Networks"
        A[NAT Network]
        B[Bridged Network]
        C[Isolated Network]
        D[Direct Passthrough]
    end
    
    subgraph "Backend"
        E[Linux Bridge]
        F[Open vSwitch]
        G[MACvtap]
    end
    
    A --> E
    B --> F
    C --> F
    D --> F
    
    style A fill:#c8e6c9
    style B fill:#bbdefb
    style C fill:#e1f5fe
    style D fill:#ff9800
    style E fill:#ffecb3
    style F fill:#a5d6a7
```

## Quick Commands

```bash
# Network management
virsh net-list
virsh net-define network.xml
virsh net-start <network-name>

# Bridge commands
brctl show
brctl addbr br0
brctl delbr br0
```
