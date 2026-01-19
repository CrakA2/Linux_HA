# Virtualization

## [QEMU](qemu.md)
Generic and open source machine emulator and virtualizer providing full-system and user-mode emulation.

## [KVM](kvm.md)
Kernel-based virtual machine for Linux providing hardware-assisted virtualization.

## [libvirt](libvirt.md)
Virtualization API and daemon for managing virtualization capabilities.

## [virsh](virsh.md)
Command-line interface for managing libvirt virtualization.

## [Open vSwitch](ovs.md)
Open source multilayer virtual switch for managing VM networking.

## Deployment Workflow

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

## Security

- Use AppArmor/SELinux profiles
- Use virtio drivers for better isolation
- Enable seccomp filters
- Use IOMMU for device passthrough
