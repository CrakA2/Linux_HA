# Cluster Technologies

Complete reference guide for Corosync and Pacemaker cluster management technologies.

## Corosync

Corosync is a Group Communication System providing virtual synchrony guarantees and quorum management for Linux HA clusters.

### Architecture Overview

```mermaid
graph TB
    A[Applications] --> B[Corosync Cluster Engine]
    B --> C[TOTEM Protocol]
    B --> D[Quorum System]
    B --> E[Configuration DB]
    B --> F[Availability Manager]
    C --> G[Network Layer]
```

### Key Features

- **Virtual Synchrony**: Consistent message ordering across all nodes
- **TOTEM Protocol**: Reliable group communication
- **Quorum Management**: Split-brain prevention
- **Configuration Database**: In-memory key-value store
- **Ring Topology**: Efficient message delivery

### Quick Commands

```bash
# Installation
apt-get install corosync

# Generate authentication key
corosync-keygen

# Status
corosync-cfgtool -s
corosync-quorumtool -s

# Configuration
corosync-cmapctl
```

### Deployment

```bash
# 1. Install and configure
apt-get install corosync
corosync-keygen
vim /etc/corosync/corosync.conf

# 2. Start on all nodes
systemctl enable corosync
systemctl start corosync

# 3. Verify cluster
corosync-cfgtool -s
```

### Source Code

- **Repository**: [corosync/corosync](https://github.com/corosync/corosync)
- **Documentation**: [corosync.github.io](https://corosync.github.io/corosync/)

### Common Issues

| Issue | Solution |
|-------|----------|
| Token timeout errors | Increase `token: 10000` in corosync.conf |
| Quorum loss | Check network connectivity and node count |
| Authentication failures | Verify authkey permissions (600) |

---

## Pacemaker

Pacemaker is an advanced cluster resource manager for Linux high availability.

### Architecture Overview

```mermaid
graph TB
    A[CRM - Cluster Resource Manager] --> B[PE - Policy Engine]
    A --> C[TE - Transition Engine]
    A --> D[LRM - Local Resource Manager]
    A --> E[CIB - Cluster Info Base]
    
    B --> F[Calculate Transitions]
    C --> G[Execute Actions]
    D --> H[Monitor Resources]
    E --> I[Cluster State Database]
```

### Key Features

- **Resource Classes**: OCF, LSB, systemd, Upstart
- **Constraints**: Ordering, colocation, location
- **STONITH**: Fencing integration
- **Cloning**: Resource and node templates
- **Batches**: Group operations

### Quick Commands

```bash
# Cluster status
pcs status

# Resource management
pcs resource create vip ocf:heartbeat:IPaddr2 ip=192.168.1.100
pcs resource start vip

# Constraints
pcs constraint order start vip then apache
pcs constraint colocation add apache with vip

# STONITH
pcs stonith create fence-device fence_ipmilan
```

### Deployment

```bash
# 1. Install components
apt-get install pacemaker pcs

# 2. Configure cluster
pcs cluster setup --name mycluster node1 node2 node3

# 3. Start cluster
pcs cluster start --all

# 4. Enable STONITH
pcs property set stonith-enabled=true
```

### Source Code

- **Repository**: [ClusterLabs/pacemaker](https://github.com/ClusterLabs/pacemaker)
- **Documentation**: [clusterlabs.org/pacemaker/doc/](https://www.clusterlabs.org/pacemaker/doc/)

### Common Issues

| Issue | Solution |
|-------|----------|
| Resource stuck starting | Check agent with `pcs resource debug-resource` |
| Constraint violations | Verify constraints with `pcs constraint show --full` |
| STONITH failures | Configure fencing devices properly |
| Quorum issues | Check network and node connectivity |

---

## Deployment Workflow

Complete Corosync and Pacemaker HA cluster setup.

### 1. Initial Setup

```bash
# Install all packages
apt-get install corosync pacemaker pcs

# Generate Corosync key
corosync-keygen

# Distribute key
scp /etc/corosync/authkey node2:/etc/corosync/
scp /etc/corosync/authkey node3:/etc/corosync/

# Configure Corosync
# Edit /etc/corosync/corosync.conf on first node

# Copy configuration to all nodes
scp /etc/corosync/corosync.conf node2:/etc/corosync/
scp /etc/corosync/corosync.conf node3:/etc/corosync/

# Start services
systemctl enable corosync pacemaker pcsd
systemctl start corosync pacemaker pcsd
```

### 2. Verify Cluster

```bash
# Check Corosync
corosync-cfgtool -s
corosync-quorumtool -s

# Check Pacemaker
pcs status
```

### 3. Configure STONITH

```bash
# Create fencing device
pcs stonith create fence-device fence_ipmilan \
    ipaddr=192.168.1.200 login=admin passwd=password

# Verify fencing
pcs stonith list
```

### 4. Add Resources

```bash
# Create VIP resource
pcs resource create vip ocf:heartbeat:IPaddr2 \
    ip=192.168.1.100 cidr_netmask=24

# Create Apache resource
pcs resource create apache ocf:heartbeat:apache \
    configfile=/etc/apache2/apache2.conf

# Add ordering constraint
pcs constraint order start vip then apache

# Add colocation constraint
pcs constraint colocation add apache with vip

# Start resources
pcs resource start vip
```

### 5. Testing

```bash
# Test resource failover
pcs resource move apache node2

# Verify cluster health
pcs status
crm_mon
```

---

## Source Code References

| Component | Repository | Documentation |
|-----------|-----------|-------------|
| Corosync | [corosync/corosync](https://github.com/corosync/corosync) | [corosync.github.io](https://corosync.github.io/corosync/) |
| Pacemaker | [ClusterLabs/pacemaker](https://github.com/ClusterLabs/pacemaker) | [clusterlabs.org/pacemaker/doc/](https://www.clusterlabs.org/pacemaker/doc/) |

---

## Nifty Behaviors

### Corosync Optimizations

- **Kronosnet**: Multi-link transport with automatic failover
- **Token tuning**: `token: 5000` for faster failure detection
- **Network buffers**: `window_size: 50` for better throughput

### Pacemaker Tips

- **Resource stickiness**: Keep resources on current node
- **Batch operations**: Apply multiple constraints together
- **Symmetric monitoring**: Enable `pacemaker` alerts

---

## Security Considerations

### Corosync
- Protect authentication key: `chmod 600 /etc/corosync/authkey`
- Use encrypted transport (knet)
- Isolate cluster network

### Pacemaker
- Always enable STONITH for production
- Limit cluster daemon permissions
- Secure cluster communication

---

---

## Corosync + Pacemaker + OVS Stack

Integrated high availability stack combining Corosync messaging, Pacemaker resource management, and Open vSwitch networking.

### Architecture Overview

```mermaid
graph TB
    A[VMs] --> B[OVS Bridge]
    B --> C[Physical Network]
    
    D[Pacemaker CRM] --> E[OVS Resources]
    D --> F[VIP Resources]
    D --> G[Application Resources]
    
    E --> B
    
    H[Corosync] --> D
    I[Quorum] --> D
    J[STONITH] --> D
    
    style B fill:#c8e6c9
    style D fill:#ffecb3
    style H fill:#e1f5ff
```

### Integration Components

- **Corosync**: Cluster messaging and quorum
- **Pacemaker**: Resource management and failover
- **OVS**: Network virtualization and connectivity

### OVS Resources in Pacemaker

```bash
# Create OVS bridge resource
pcs resource create ovs-bridge ocf:heartbeat:ovs-bridge \
    bridge_name=br0

# Create OVS port resource
pcs resource create ovs-port ocf:heartbeat:ovs-port \
    bridge_name=br0 port_name=eth0

# Create OVS interface resource
pcs resource create ovs-intf ocf:heartbeat:ovs-interface \
    bridge_name=br0 interface=eth1 ip=192.168.1.100/24

# Add ordering constraints
pcs constraint order start ovs-bridge then ovs-port
pcs constraint order start ovs-port then ovs-intf

# Add colocation constraints
pcs constraint colocation add ovs-port with ovs-bridge
pcs constraint colocation add ovs-intf with ovs-port
```

### Full Stack Deployment

```bash
# 1. Install all components
apt-get install corosync pacemaker pcs openvswitch-switch

# 2. Configure Corosync
corosync-keygen
vim /etc/corosync/corosync.conf

# 3. Configure Pacemaker cluster
pcs cluster setup --name ha-cluster node1 node2 node3
pcs cluster start --all

# 4. Configure OVS
ovs-vsctl add-br br0
ovs-vsctl add-port br0 eth0
ovs-vsctl add-port br0 eth1

# 5. Create OVS resources
pcs resource create ovs-bridge ocf:heartbeat:ovs-bridge bridge_name=br0
pcs resource create ovs-port ocf:heartbeat:ovs-port bridge_name=br0 port_name=eth0
pcs resource create ovs-intf ocf:heartbeat:ovs-interface bridge_name=br0 interface=eth1 ip=192.168.1.100/24

# 6. Create VIP resource
pcs resource create vip ocf:heartbeat:IPaddr2 ip=192.168.1.200 cidr_netmask=24

# 7. Create application resource
pcs resource create app systemd:myapp

# 8. Add constraints
pcs constraint order start ovs-bridge then ovs-port
pcs constraint order start ovs-port then ovs-intf
pcs constraint order start ovs-intf then vip
pcs constraint order start vip then app

pcs constraint colocation add ovs-port with ovs-bridge
pcs constraint colocation add ovs-intf with ovs-port
pcs constraint colocation add vip with ovs-intf
pcs constraint colocation add app with vip

# 9. Enable STONITH
pcs stonith create fence-device fence_ipmilan
pcs property set stonith-enabled=true

# 10. Start resources
pcs resource start ovs-bridge
```

### Sample Corosync Configuration

```conf
totem {
    version: 2
    cluster_name: ha-cluster
    transport: knet
    interface {
        ringnumber: 0
        bindnetaddr: 192.168.1.0
        mcastport: 5405
    }
}

nodelist {
    node {
        ring0_addr: 192.168.1.10
        name: node1
    }
    node {
        ring0_addr: 192.168.1.11
        name: node2
    }
    node {
        ring0_addr: 192.168.1.12
        name: node3
    }
}

quorum {
    provider: corosync_votequorum
    expected_votes: 3
}

logging {
    to_logfile: yes
    logfile: /var/log/corosync/corosync.log
    to_syslog: yes
}
```

### Resource Monitoring

```bash
# Monitor all resources
pcs status resources

# Monitor OVS-specific resources
pcs status resources ovs-bridge ovs-port ovs-intf

# Check OVS configuration
ovs-vsctl show

# Check Corosync status
corosync-cfgtool -s
corosync-quorumtool -s

# Check Pacemaker status
pcs status
crm_mon
```

### Failover Testing

```bash
# Test OVS failover
pcs resource move ovs-bridge node2

# Test application failover
pcs resource move app node3

# Verify network connectivity
ping 192.168.1.200

# Verify OVS bridge
ovs-ofctl show br0
```

### Common Issues

| Issue | Solution |
|-------|----------|
| OVS bridge not starting | Check OVS agent logs: `pcs resource debug-resource ovs-bridge` |
| VIP not accessible | Verify OVS bridge is running before VIP |
| Network partition | Verify Corosync quorum: `corosync-quorumtool -s` |
| Resource stuck | Check constraints: `pcs constraint show --full` |

### Nifty Behaviors

### OVS Bridge Resource Clone
```bash
pcs resource create ovs-bridge ocf:heartbeat:ovs-bridge bridge_name=br0 \
    --clone
```
**Nifty**: Run OVS bridge on all nodes simultaneously

### Symmetric Cloning for High Availability
```bash
pcs resource clone app clone-max=2 clone-node-max=1
```
**Nifty**: Application runs on multiple nodes with load balancing

### OVS Interface with Bonding
```bash
ovs-vsctl add-bond bond0 eth0 eth1
pcs resource create ovs-bond ocf:heartbeat:ovs-port \
    bridge_name=br0 port_name=bond0
```
**Nifty**: Network redundancy with bonded interfaces

---

## Troubleshooting

For in-depth troubleshooting focused on code behavior and diagnostics, see [Deployment](../deployment.md) section.
