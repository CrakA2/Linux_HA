# Quick Reference

## Corosync

| Command | Description |
|---------|-------------|
| `corosync-cfgtool -s` | Show cluster status |
| `corosync-quorumtool -s` | Show quorum status |
| `corosync-cmapctl` | Query configuration |
| `corosync-keygen` | Generate auth key |

## Pacemaker

| Command | Description |
|---------|-------------|
| `pcs status` | Display cluster status |
| `pcs resource create` | Create resource |
| `pcs constraint order` | Add ordering |
| `pcs constraint colocation` | Add colocation |
| `pcs stonith create` | Create STONITH |

## QEMU/KVM

| Command | Description |
|---------|-------------|
| `qemu-system-x86_64` | Run QEMU VM |
| `virsh list` | List VMs |
| `virsh start` | Start VM |
| `virsh shutdown` | Shutdown VM |
| `kvm-ok` | Check KVM support |

## CEPH

| Command | Description |
|---------|-------------|
| `ceph -s` | Cluster status |
| `ceph health` | Health check |
| `rbd create` | Create RBD image |
| `rbd map` | Map RBD image |
| `ceph orch apply` | Apply service |

## iSCSI

| Command | Description |
|---------|-------------|
| `iscsiadm -m discovery` | Discover targets |
| `iscsiadm -m session` | List sessions |
| `gwcli.py target list` | List targets |
| `rbd showmapped` | Show mapped RBDs |

## NVMe-oF

| Command | Description |
|---------|-------------|
| `nvme list` | List NVMe devices |
| `nvme discover` | Discover subsystems |
| `gwcli.py subsystem list` | List subsystems |
| `nvme show-ctrl` | Show controller |

## GFS2

| Command | Description |
|---------|-------------|
| `gfs2_tool sb` | Show superblock |
| `gfs2_tool df` | Show usage |
| `dlm_tool ls` | List DLM nodes |
| `dlm_tool dump` | Dump locks |

## Open vSwitch

| Command | Description |
|---------|-------------|
| `ovs-vsctl add-br` | Add bridge |
| `ovs-vsctl del-br` | Delete bridge |
| `ovs-vsctl show` | Show config |
| `ovs-ofctl show` | Show OpenFlow rules |
