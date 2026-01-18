# FC (NVMe-oF) with CEPH

## Table of Contents
1. [Introduction and Architecture](#1-introduction-and-architecture)
2. [Core Components](#2-core-components)
3. [Deployment Recommendations](#3-deployment-recommendations)
4. [Code Behavior Analysis](#4-code-behavior-analysis)
5. [Source Code Reference](#5-source-code-reference)
6. [Configuration and Management](#6-configuration-and-management)
7. [Troubleshooting and Monitoring](#7-troubleshooting-and-monitoring)
8. [Advanced Topics](#8-advanced-topics)

See main course material: [../fc-nvmeof-ceph-course-material.md](../fc-nvmeof-ceph-course-material.md)

## Quick Reference

| Command | Description |
|---------|-------------|
| `gwcli.py subsystem list` | List NVMe subsystems |
| `nvme discover` | Discover subsystems |
| `nvme list` | List NVMe devices |
| `nvme list-subsys` | List subsystems |
| `nvme show-ctrl /dev/nvme0` | Show controller |
| `ceph orch apply nvmeof gateway.yml` | Deploy NVMe-oF gateway |

## Source Code
- Ceph NVMe-oF Docs: https://docs.ceph.com/en/latest/rbd/nvmeof-overview/
- NVMe Express: https://nvmexpress.org/
- SPDK: https://www.spdk.io/
- Source Code: https://github.com/ceph/ceph
