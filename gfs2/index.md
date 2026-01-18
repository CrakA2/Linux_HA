# GFS2

## Table of Contents
1. [Introduction and Architecture](#1-introduction-and-architecture)
2. [Core Components](#2-core-components)
3. [Deployment Recommendations](#3-deployment-recommendations)
4. [Code Behavior Analysis](#4-code-behavior-analysis)
5. [Source Code Reference](#5-source-code-reference)
6. [Configuration and Management](#6-configuration-and-management)
7. [Troubleshooting and Monitoring](#7-troubleshooting-and-monitoring)
8. [Advanced Topics](#8-advanced-topics)

See main course material: [../gfs2-course-material.md](../gfs2-course-material.md)

## Quick Reference

| Command | Description |
|---------|-------------|
| `gfs2_tool sb` | Show superblock |
| `gfs2_tool df` | Show filesystem usage |
| `gfs2_quota enable` | Enable quota |
| `dlm_tool ls` | List DLM nodes |
| `dlm_tool dump` | Dump lock table |

## Source Code
- Documentation: /usr/share/doc/gfs2-utils/
- Source Code: https://github.com/torvalds/linux/tree/master/fs/gfs2
- Pacemaker Docs: https://www.clusterlabs.org/pacemaker/doc/
