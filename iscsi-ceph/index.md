# iSCSI with CEPH

## Table of Contents
1. [Introduction and Architecture](#1-introduction-and-architecture)
2. [Core Components](#2-core-components)
3. [Deployment Recommendations](#3-deployment-recommendations)
4. [Code Behavior Analysis](#4-code-behavior-analysis)
5. [Source Code Reference](#5-source-code-reference)
6. [Configuration and Management](#6-configuration-and-management)
7. [Troubleshooting and Monitoring](#7-troubleshooting-and-monitoring)
8. [Advanced Topics](#8-advanced-topics)

See main course material: [../iscsi-ceph-course-material.md](../iscsi-ceph-course-material.md)

## Quick Reference

| Command | Description |
|---------|-------------|
| `gwcli.py target list` | List iSCSI targets |
| `iscsiadm -m discovery` | Discover targets |
| `iscsiadm -m session` | List sessions |
| `rbd map` | Map RBD image |
| `rbd showmapped` | Show mapped RBDs |
| `ceph orch apply iscsi gateway.yml` | Deploy iSCSI gateway |

## Source Code
- Ceph iSCSI Docs: https://docs.ceph.com/en/latest/rbd/iscsi-overview/
- LIO Documentation: https://linux-iscsi.org/wiki/HomePage
- Source Code: https://github.com/ceph/ceph
