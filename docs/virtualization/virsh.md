# virsh

Command-line interface for managing libvirt virtualization.

## Quick Commands

```bash
# VM Management
virsh list                   # List running VMs
virsh list --all            # List all VMs
virsh start <vm-name>        # Start VM
virsh shutdown <vm-name>     # Shutdown VM
virsh destroy <vm-name>      # Force stop VM
virsh undefine <vm-name>    # Remove VM

# VM Information
virsh dominfo <vm-name>      # Domain information
virsh dumpxml <vm-name>      # XML configuration
virsh vcpuinfo <vm-name>     # VCPU information
virsh dommemstat <vm-name>   # Memory statistics

# Snapshot Management
virsh snapshot-create <vm-name> <snap-name>
virsh snapshot-list <vm-name>
virsh snapshot-revert <vm-name> <snap-name>

# Console Access
virsh console <vm-name>
virsh qemu-monitor-command <vm-name> "info status"
```
