# STONITH and Fencing

Comprehensive guide to STONITH (Shoot The Other Node In The Head) and fencing for Linux HA clusters.

## Overview

STONITH is the fencing mechanism used by Pacemaker to prevent split-brain scenarios by ensuring failed nodes are properly fenced.

```mermaid
graph TB
    subgraph "Cluster Layer"
        A[Pacemaker]
        B[STONITH Resources]
        C[Quorum]
    end
    
    subgraph "Fencing Layer"
        D[Fence Agents]
        E[Fence Device]
        F[Network]
    end
    
    subgraph "Hardware Layer"
        G[IPMI/iLO/DRAC]
        H[Network Switches]
        I[PDU - Power Distribution Units]
    end
    
    subgraph "Cloud Layer"
        J[AWS EC2]
        K[OpenStack Nova]
        L[Azure VM]
    end
    
    A --> B
    A --> C
    
    B --> D
    D --> E
    E --> F
    
    E --> G
    E --> H
    E --> I
    
    E --> J
    E --> K
    E --> L
    
    style B fill:#c8e6c9
    style D fill:#ffecb3
    style G fill:#e1f5ff
    style J fill:#fff3e0
```

## STONITH Architecture

### Fencing Process

```mermaid
sequenceDiagram
    participant C as Cluster
    participant S as STONITH Daemon
    participant F as Fence Agent
    participant H as Hardware
    
    C->>S: Node failure detected
    S->>F: Initiate fence
    F->>H: Send fence command
    H->>H: Execute fence (reboot/poweroff)
    H->>F: Fence complete
    F->>S: Fence success
    S->>C: Node fenced
    C->>C: Recover resources
    
    style S fill:#c8e6c9
    style F fill:#ffecb3
    style H fill:#e1f5ff
```

### STONITH Components

| Component | Description |
|-----------|-------------|
| STONITH Daemon | Manages fencing operations |
| Fence Agents | Scripts for fencing devices |
| Fence Resources | Cluster resources for fencing |
| Fence Levels | Primary/secondary fencing |
| Fence Methods | Multiple fencing strategies |

## Fence Agent Types

### IPMI Fence Agents

**fence_ipmilan**: Intel IPMI LAN interface

```bash
# Create IPMI fence device
pcs stonith create fence-ipmi-node1 \
    fence_ipmilan \
    ipaddr=192.168.1.201 \
    login=admin \
    passwd=AdminPass123 \
    lanplus=1 \
    action=reboot \
    pcmk_host_list=node1
```

**Parameters**:
| Parameter | Description | Default |
|-----------|-------------|---------|
| ipaddr | IPMI IP address | - |
| login | IPMI username | - |
| passwd | IPMI password | - |
| port | IPMI port | 623 |
| lanplus | Use IPMIv2.0 | 0 |
| action | Fencing action (reboot/off/on) | reboot |
| timeout | Timeout in seconds | 20 |
| cipher | Cipher suite (0-15) | 17 |

### iLO Fence Agents

**fence_ilo**: HP iLO fencing

```bash
# Create iLO fence device
pcs stonith create fence-ilo-node1 \
    fence_ilo \
    ipaddr=192.168.1.202 \
    login=admin \
    passwd=AdminPass123 \
    action=reboot \
    pcmk_host_list=node1
```

**Parameters**:
| Parameter | Description | Default |
|-----------|-------------|---------|
| ipaddr | iLO IP address | - |
| login | iLO username | - |
| passwd | iLO password | - |
| ssl | Use SSL | 1 |
| port | iLO port | 443 |
| action | Fencing action | reboot |
| timeout | Timeout in seconds | 20 |

### DRAC Fence Agents

**fence_drac5**: Dell DRAC 5 fencing

```bash
# Create DRAC fence device
pcs stonith create fence-drac-node1 \
    fence_drac5 \
    ipaddr=192.168.1.203 \
    login=root \
    passwd=AdminPass123 \
    module=1 \
    action=reboot \
    pcmk_host_list=node1
```

**Parameters**:
| Parameter | Description | Default |
|-----------|-------------|---------|
| ipaddr | DRAC IP address | - |
| login | DRAC username | - |
| passwd | DRAC password | - |
| module | DRAC module number | 1 |
| cmd_prompt | Command prompt | (\[A-Za-z0-9\])+\s*>? |
| action | Fencing action | reboot |

### VMware Fence Agents

**fence_vmware_soap**: VMware vSphere fencing

```bash
# Create VMware fence device
pcs stonith create fence-vmware \
    fence_vmware_soap \
    ipaddr=192.168.1.210 \
    login=administrator \
    passwd=AdminPass123 \
    ssl=1 \
    ipport=443 \
    action=reboot \
    pcmk_host_list="node1 node2 node3"
```

**Parameters**:
| Parameter | Description | Default |
|-----------|-------------|---------|
| ipaddr | vCenter IP address | - |
| login | vCenter username | - |
| passwd | vCenter password | - |
| ssl | Use SSL | 1 |
| ipport | vCenter port | 443 |
| vm_name | VM name pattern | - |
| action | Fencing action | reboot |

### AWS Fence Agents

**fence_aws**: Amazon AWS fencing

```bash
# Create AWS fence device
pcs stonith create fence-aws \
    fence_aws \
    region=us-east-1 \
    access-key=AKIAIOSFODNN7EXAMPLE \
    secret-key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY \
    action=reboot
```

**Parameters**:
| Parameter | Description | Default |
|-----------|-------------|---------|
| region | AWS region | - |
| access-key | AWS access key | - |
| secret-key | AWS secret key | - |
| profile | AWS profile | - |
| action | Fencing action | reboot |

### PDU Fence Agents

**fence_apc**: APC PDU fencing

```bash
# Create APC PDU fence device
pcs stonith create fence-apc-pdu \
    fence_apc \
    ipaddr=192.168.1.220 \
    login=admin \
    passwd=AdminPass123 \
    port=1 \
    action=reboot \
    pcmk_host_list=node1
```

**Parameters**:
| Parameter | Description | Default |
|-----------|-------------|---------|
| ipaddr | PDU IP address | - |
| login | PDU username | - |
| passwd | PDU password | - |
| port | Outlet number | - |
| switch | Switch number | 1 |
| action | Fencing action | reboot |

### Switch Fence Agents

**fence_ifmib**: SNMP-based fencing

```bash
# Create SNMP fence device
pcs stonith create fence-switch \
    fence_ifmib \
    ipaddr=192.168.1.230 \
    community=public \
    action=reboot
```

**Parameters**:
| Parameter | Description | Default |
|-----------|-------------|---------|
| ipaddr | Switch IP address | - |
| community | SNMP community | public |
| port | Port number | - |
| action | Fencing action | reboot |

## STONITH Configuration

### Basic Configuration

```bash
# Enable STONITH
pcs property set stonith-enabled=true

# Set STONITH timeout
pcs property set stonith-timeout=60s

# Set STONITH action
pcs property set stonith-action=reboot

# Set STONITH failure handling
pcs property set stonith-failure-is-fatal=true
```

### Per-Node Fencing

```bash
# IPMI fencing for node1
pcs stonith create fence-ipmi-node1 \
    fence_ipmilan \
    ipaddr=192.168.1.201 \
    login=admin \
    passwd=AdminPass123 \
    pcmk_host_list=node1

# IPMI fencing for node2
pcs stonith create fence-ipmi-node2 \
    fence_ipmilan \
    ipaddr=192.168.1.202 \
    login=admin \
    passwd=AdminPass123 \
    pcmk_host_list=node2

# IPMI fencing for node3
pcs stonith create fence-ipmi-node3 \
    fence_ipmilan \
    ipaddr=192.168.1.203 \
    login=admin \
    passwd=AdminPass123 \
    pcmk_host_list=node3
```

### Shared Fencing Device

```bash
# Single fence device for all nodes
pcs stonith create fence-vmware \
    fence_vmware_soap \
    ipaddr=192.168.1.210 \
    login=administrator \
    passwd=AdminPass123 \
    pcmk_host_list="node1 node2 node3"
```

### Fence Levels

```bash
# Primary fencing level (IPMI)
pcs stonith level add 1 node1 fence-ipmi-node1

# Secondary fencing level (VMware)
pcs stonith level add 2 node1 fence-vmware

# View fence levels
pcs stonith level

# Remove fence level
pcs stonith level remove 1 node1
```

### Fence Methods

```bash
# Define fence method
pcs stonith create fence-ipmi-ilo \
    fence_ipmilan \
    ipaddr=192.168.1.201 \
    login=admin \
    passwd=AdminPass123 \
    pcmk_host_list=node1 \
    op monitor interval=60s

pcs stonith create fence-ilo-fallback \
    fence_ilo \
    ipaddr=192.168.1.202 \
    login=admin \
    passwd=AdminPass123 \
    pcmk_host_list=node1
```

## STONITH Operations

### Testing Fencing

```bash
# Test fencing manually
fence_ipmilan -a 192.168.1.201 -l admin -p AdminPass123 \
    -o status

# Test fencing via pcs
pcs stonith fence node1

# Test fencing with verbose output
pcs stonith fence node1 --verbose
```

### Monitoring Fencing

```bash
# Check STONITH status
pcs stonith status

# Check STONITH devices
pcs stonith show

# Check fence device details
pcs stonith show fence-ipmi-node1

# Check STONITH logs
journalctl -u pacemaker -f | grep stonith
```

### Fence Device Management

```bash
# Start fence device
pcs resource start fence-ipmi-node1

# Stop fence device
pcs resource stop fence-ipmi-node1

# Enable fence device
pcs resource enable fence-ipmi-node1

# Disable fence device
pcs resource disable fence-ipmi-node1

# Delete fence device
pcs resource delete fence-ipmi-node1
```

## Advanced Configuration

### STONITH Properties

| Property | Description | Default |
|----------|-------------|---------|
| stonith-enabled | Enable/disable STONITH | false |
| stonith-timeout | Timeout for fencing | 60s |
| stonith-action | Default fencing action | reboot |
| stonith-failure-is-fatal | Stop cluster on fence failure | true |
| stonith-watchdog-timeout | Watchdog timeout | 0 |
| stonith-max-attempts | Max fencing attempts | 2 |

### Fence Device Attributes

```bash
# Set timeout
pcs resource update fence-ipmi-node1 \
    meta target-role=Started

# Set monitoring interval
pcs resource op add fence-ipmi-node1 \
    monitor interval=60s timeout=90s

# Set resource stickiness
pcs resource update fence-ipmi-node1 \
    meta resource-stickiness=0
```

### Fence Device Constraints

```bash
# Colocate fencing with node
pcs constraint colocation add fence-ipmi-node1 with node1

# Order fencing before resources
pcs constraint order start fence-ipmi-node1 then apache

# Location constraint
pcs constraint location fence-ipmi-node1 prefers node1
```

## Troubleshooting

### Fencing Failures

**Check fence device status**:
```bash
# Check STONITH status
pcs stonith show

# Check resource status
pcs status resources

# Check cluster status
pcs status
```

**Test fence device manually**:
```bash
# Test IPMI fencing
fence_ipmilan -a 192.168.1.201 -l admin -p AdminPass123 \
    -o status

# Test iLO fencing
fence_ilo -a 192.168.1.202 -l admin -p AdminPass123 \
    -o status

# Test DRAC fencing
fence_drac5 -a 192.168.1.203 -l root -p AdminPass123 \
    -o status
```

**Check logs**:
```bash
# Check Pacemaker logs
journalctl -u pacemaker -f

# Check STONITH logs
grep stonith /var/log/pacemaker/pacemaker.log

# Check fence agent logs
journalctl -f | grep fence
```

### Network Issues

**Check connectivity**:
```bash
# Ping fence device
ping -c 3 192.168.1.201

# Check port accessibility
telnet 192.168.1.201 623
nc -zv 192.168.1.201 623
```

**Check firewall**:
```bash
# Check firewall rules
iptables -L -n

# Open IPMI port
iptables -A INPUT -p tcp --dport 623 -j ACCEPT
iptables -A INPUT -p udp --dport 623 -j ACCEPT

# Check UFW (Ubuntu)
ufw allow 623/tcp
ufw allow 623/udp
```

### Authentication Issues

**Verify credentials**:
```bash
# Test IPMI credentials
ipmitool -I lanplus -H 192.168.1.201 \
    -U admin -P AdminPass123 chassis status

# Test iLO credentials
curl -k https://192.168.1.202/ \
    --user admin:AdminPass123

# Test DRAC credentials
racadm -r 192.168.1.203 -u root -p AdminPass123 \
    getinfo
```

**Check fence device configuration**:
```bash
# View fence device configuration
pcs stonith show fence-ipmi-node1

# Check fence device status
pcs resource show fence-ipmi-node1
```

## Cloud Fencing

### AWS EC2 Fencing

```bash
# Create AWS fence device
pcs stonith create fence-aws \
    fence_aws \
    region=us-east-1 \
    access-key=AKIAIOSFODNN7EXAMPLE \
    secret-key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# Map instances to fence device
pcs stonith create fence-aws-node1 \
    fence_aws \
    region=us-east-1 \
    access-key=AKIAIOSFODNN7EXAMPLE \
    secret-key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY \
    pcmk_host_map="node1:i-1234567890abcdef0"
```

### OpenStack Nova Fencing

```bash
# Create OpenStack fence device
pcs stonith create fence-openstack \
    fence_openstack \
    auth-url=http://192.168.1.200:5000/v3 \
    username=admin \
    password=AdminPass123 \
    project-id=service \
    domain-id=default \
    region-name=RegionOne

# Map instances
pcs stonith create fence-openstack-node1 \
    fence_openstack \
    auth-url=http://192.168.1.200:5000/v3 \
    username=admin \
    password=AdminPass123 \
    project-id=service \
    domain-id=default \
    region-name=RegionOne \
    pcmk_host_map="node1:server-1-uuid"
```

## Best Practices

1. **Always enable STONITH** in production
2. **Use redundant fencing** with multiple levels
3. **Test fencing devices** regularly
4. **Monitor fencing operations**
5. **Use appropriate timeout** values
6. **Secure fence credentials**
7. **Document fencing configuration**
8. **Use fence device monitoring**
9. **Test fencing scenarios**
10. **Have manual fencing procedure ready**

## Fencing Strategies

### IPMI + VMware

```bash
# Primary: IPMI (fast)
pcs stonith level add 1 node1 fence-ipmi-node1

# Secondary: VMware (reliable)
pcs stonith level add 2 node1 fence-vmware
```

### iLO + DRAC

```bash
# Primary: iLO
pcs stonith level add 1 node1 fence-ilo-node1

# Secondary: DRAC
pcs stonith level add 2 node1 fence-drac-node1
```

### PDU + Network

```bash
# Primary: PDU (power off)
pcs stonith level add 1 node1 fence-apc-pdu

# Secondary: Network switch (disconnect)
pcs stonith level add 2 node1 fence-switch
```

## Security

### Secure Fence Credentials

```bash
# Use Pacemaker CIB secrets
pcs resource secret set fence-ipmi-node1 passwd

# Or use environment variables
export FENCE_PASSWD="AdminPass123"
pcs stonith create fence-ipmi-node1 \
    fence_ipmilan \
    ipaddr=192.168.1.201 \
    login=admin \
    passwd="${FENCE_PASSWD}"
```

### Network Security

```bash
# Use dedicated network for fencing
ip route add 192.168.100.0/24 dev eth1

# Use firewall rules
iptables -A INPUT -s 192.168.100.0/24 -p tcp --dport 623 -j ACCEPT
iptables -A INPUT -p tcp --dport 623 -j DROP

# Use IPsec for fencing traffic
ip xfrm state add src 192.168.100.0/24 dst 192.168.1.0/24 proto esp spi 0x200 mode transport reqid 1000
```

## Monitoring

### Fencing Alerts

```bash
# Monitor fencing events
crm_mon --one-shot --as-xml | grep stonith

# Monitor fence device status
pcs stonith show --full

# Monitor fencing logs
tail -f /var/log/pacemaker/pacemaker.log | grep stonith
```

### Fencing Metrics

```bash
# Check fence device performance
pcs resource show fence-ipmi-node1

# Check fencing history
crm_history --show fence-ipmi-node1

# Check fencing statistics
pcs status resources | grep fence
```

## Fencing Simulation

```bash
# Simulate fencing (without actually fencing)
pcs stonith fence node1 --dry-run

# Simulate fencing failure
pcs stonith fence node1 --timeout 1

# Simulate network partition
iptables -A INPUT -s 192.168.1.0/24 -j DROP
```

## Recovery Procedures

### Manual Fencing

```bash
# Manually fence node via IPMI
ipmitool -I lanplus -H 192.168.1.201 -U admin -P AdminPass123 \
    power cycle

# Manually fence node via iLO
curl -k -X POST https://192.168.1.202/rest/v1/Systems/1/Actions/ComputerSystem.Reset \
    -d '{"ResetType":"ForceRestart"}' \
    --user admin:AdminPass123

# Manually fence node via DRAC
racadm -r 192.168.1.203 -u root -p AdminPass123 serveraction graceful
```

### Recovery After Fencing

```bash
# Wait for node to reboot
sleep 60

# Check cluster status
pcs status

# Verify node rejoined cluster
pcs status nodes

# Start resources on node
pcs resource move apache node1
```

## Source Code

- **Fence Agents**: https://github.com/ClusterLabs/fence-agents
- **Pacemaker**: https://github.com/ClusterLabs/pacemaker
- **STONITH Documentation**: https://www.clusterlabs.org/pacemaker/doc/2.0/Pacemaker_Explained/

### Key Fence Agent Locations

| Agent | Location | Description |
|-------|----------|-------------|
| fence_ipmilan | `agents/ipmilan/fence_ipmilan.py` | IPMI LAN |
| fence_ilo | `agents/ilo/fence_ilo.py` | HP iLO |
| fence_drac5 | `agents/drac5/fence_drac5.py` | Dell DRAC |
| fence_vmware_soap | `agents/vmware_soap/fence_vmware_soap.py` | VMware vSphere |
| fence_aws | `agents/aws/fence_aws.py` | Amazon AWS |
| fence_apc | `agents/apc/fence_apc.py` | APC PDU |