# QEMU and KVM

## Table of Contents
1. [Introduction and Architecture](#1-introduction-and-architecture)
2. [QEMU Architecture](#2-qemu-architecture)
3. [KVM Architecture](#3-kvm-architecture)
4. [Integration of QEMU and KVM](#4-integration-of-qemu-and-kvm)
5. [Deployment Recommendations](#5-deployment-recommendations)
6. [Code Behavior Analysis](#6-code-behavior-analysis)
7. [Source Code Reference](#7-source-code-reference)
8. [Virtualization Modes](#8-virtualization-modes)
9. [Configuration and Management](#9-configuration-and-management)
10. [Troubleshooting and Monitoring](#10-troubleshooting-and-monitoring)

See main course material: [../qemu-kvm-course-material.md](../qemu-kvm-course-material.md)

## Quick Reference

| Command | Description |
|---------|-------------|
| `virsh list` | List VMs |
| `virsh start` | Start VM |
| `virsh shutdown` | Shutdown VM |
| `virsh dumpxml` | Show VM configuration |
| `qemu-system-x86_64` | Run QEMU VM |

## Source Code
- QEMU: https://gitlab.com/qemu-project/qemu
- KVM: https://github.com/torvalds/linux/tree/master/virt/kvm
- Documentation: https://www.qemu.org/documentation/
