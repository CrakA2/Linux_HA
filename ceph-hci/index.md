# CEPH Hyper-Converged Infrastructure (HCI)

## Table of Contents
1. [Introduction and Architecture](#1-introduction-and-architecture)
2. [Core Components](#2-core-components)
3. [Deployment Recommendations](#3-deployment-recommendations)
4. [Code Behavior Analysis](#4-code-behavior-analysis)
5. [Source Code Reference](#5-source-code-reference)
6. [Cephadm Deployment](#6-cephadm-deployment)
7. [Configuration and Management](#7-configuration-and-management)
8. [Troubleshooting and Monitoring](#8-troubleshooting-and-monitoring)
9. [Advanced Topics](#9-advanced-topics)

See main course material: [../ceph-hci-course-material.md](../ceph-hci-course-material.md)

## Quick Reference

| Command | Description |
|---------|-------------|
| `ceph -s` | Cluster status |
| `ceph health` | Health check |
| `ceph osd tree` | OSD tree |
| `rbd create` | Create RBD image |
| `rbd map` | Map RBD image |
| `cephadm bootstrap` | Bootstrap cluster |
| `ceph orch apply` | Apply services |

## Source Code
- Repository: https://github.com/ceph/ceph
- Documentation: https://docs.ceph.com/
- Cephadm: https://docs.ceph.com/en/latest/cephadm/install/
