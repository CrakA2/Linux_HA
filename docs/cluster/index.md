# Cluster Technologies

## [Corosync](corosync.md)

Cluster messaging and synchronization framework providing virtual synchrony guarantees and quorum management.

## [Pacemaker](pacemaker.md)

Advanced cluster resource manager coordinating configuration, start-up, monitoring, and recovery of interrelated services.

## Deployment Workflow

```bash
# 1. Install components
apt-get install corosync pacemaker pcs

# 2. Generate keys
corosync-keygen

# 3. Configure Corosync
vim /etc/corosync/corosync.conf

# 4. Start cluster
systemctl enable corosync pacemaker pcsd
systemctl start corosync pacemaker pcsd

# 5. Configure Pacemaker
pcs cluster setup --name mycluster node1 node2 node3
pcs cluster start --all
```
