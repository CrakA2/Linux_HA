## Troubleshooting Code Behavior and Diagnostics

This section provides comprehensive guidance on diagnosing and resolving issues with clustering, virtualization, and storage technologies from a code behavior perspective.

### Debugging Techniques

#### Enable Debug Logging

**Corosync:**
```bash
# Enable debug mode in corosync.conf
logging {
    to_logfile: yes
    logfile: /var/log/corosync/corosync.log
    debug: on
    debug_logfile: /var/log/corosync/debug.log
}

# Or use cormapctl
corosync-cmapctl totem.debug = 1
corosync-cmapctl totem.logging_debug = 1
```

**Pacemaker:**
```bash
# Enable verbose logging
crm_simulate --live-check
crm_simulate --show-failcounts
crm_simulate --showscores

# Check detailed resource actions
pcs resource show-status

# Monitor CIB changes
crm_mon --show-detail
crm_mon --show-node-attributes
```

**QEMU/KVM:**
```bash
# Enable QEMU debug output
qemu-system-x86_64 -d in_asm,op,exec,cpu_reset,guest_errors,int,mmu

# QMP debugging
qemu-system-x86_64 -qmp stdio -monitor stdio

# Enable KVM debug
echo 1 > /sys/module/kvm/parameters/debug
echo 1 > /sys/module/kvm_intel/parameters/debug
echo 1 > /sys/module/kvm_amd/parameters/debug
```

**Ceph:**
```bash
# Enable debug logging
ceph config set global debug_msgr 10
ceph config set mon debug_mon 10

# Enable object logging
ceph config set global debug_filestore 20
ceph config set global debug_rados 20

# Monitor OSD operations
ceph tell osd.0 dump_ops
```

### Common Code-Level Issues

#### Corosync Issues

**1. Token Circulation Problems**

```
Symptoms:
- Nodes unable to send messages
- Token stuck on one node
- Messages delayed excessively

Causes:
- Network partition
- Token timeout too short
- Network buffer overflow

Diagnosis:
corosync-cfgtool -s
corosync-quorumtool -s

Check token status:
corosync-cmapctl runtime.totem.token

Check ring state:
corosync-cmapctl runtime.totem.token_hold_time
```

```
Solution:
```
# Increase token timeout
totem {
    token: 10000           # Increase from default 10000
    token_retransmit: 500
}

# Reduce network traffic
totem {
    window_size: 50        # Reduce from default 100
    max_messages: 17
}

# Check network configuration
ping -c 3 <target-host>
tcpdump -i any 'port 5405'
```

**2. Quorum Failures**

```
Symptoms:
- Cluster loses quorum
- Nodes operate independently
- Resources stop responding

Diagnosis:
corosync-quorumtool -s

Check expected vs actual votes
Check node connectivity
```

```
Solution:
```
# Verify network connectivity
ping -c 5 <all-cluster-nodes>

# Check Corosync communication
corosync-cfgtool -s

# Adjust quorum settings
nodelist {
    node {
        quorum_votes: 3
    }
}

# Ensure proper node count
# Use odd number of nodes for majority
```

#### Pacemaker Issues

**1. Resource Start Failures**

```
Symptoms:
- Resources stuck in "Starting" state
- Resources fail to start repeatedly
- CRM crashes or restarts

Diagnosis:
pcs status

Check resource status:
pcs resource show-status resource-name

Check CRM logs:
journalctl -u pacemaker -n 50 | less

Check transition history:
crm_history --show
```

```
Solution:
```
# Increase operation timeout
pcs resource op add resource-name start \
    timeout=60s interval=0s on-fail=restart

# Check resource agent
pcs resource debug-resource resource-name
crm_resource --validate --resource resource-name

# Check for configuration errors
crm_verify -L -V
```

**2. Constraint Violations**

```
Symptoms:
- Resources not placed according to constraints
- Unexpected resource placement
- Colocation not respected

Diagnosis:
pcs constraint show --full

Check constraint scores:
crm_simulate --showscores

Verify constraint definitions:
pcs constraint order list
pcs constraint colocation list
```

```
Solution:
```
# Verify constraints are correct
pcs constraint order list
pcs constraint colocation list

# Adjust constraint scores
pcs constraint order set order-id \
    symmetrical=false

# Clear broken constraints
pcs constraint remove constraint-id

# Recalculate cluster state
pcs resource cleanup
```

**3. Resource Agent Bugs**

```
Symptoms:
- OCF agent returns invalid status codes
- Agent timeout during start/stop
- Agent crashes

Diagnosis:
# Test agent manually
ocf_resource:ocf:heartbeat:IPaddr2 monitor

# Check agent logs
grep "resource-name" /var/log/pacemaker/pengine/*

# Validate agent syntax
crm_resource --validate --resource resource-name
```

```
Solution:
```
# Test agent manually
export OCF_ROOT=/usr/lib/ocf/resource.d/heartbeat
/usr/lib/ocf/resource.d/heartbeat/IPaddr2 monitor
OCF_RESKEY=OCF_ROOT=/usr/lib/ocf/resource.d/heartbeat

# Update agent operation timeout
pcs resource op add resource-name start \
    timeout=120s

# Monitor agent behavior
pcs resource op monitor resource-name \
    timeout=30s interval=10s

# Use standard agents if possible
# Ensure OCF-compliant agents
```

#### QEMU/KVM Issues

**1. VM Startup Failures**

```
Symptoms:
- VM fails to boot
- Kernel panic in guest
- No console output

Diagnosis:
qemu-system-x86_64 -d in_asm,cpu_reset

# Check VM configuration
virsh dumpxml vm-name

# Monitor QMP events
echo '{"execute":"qmp_capabilities"}' | \
    socat UNIX-CONNECT:/var/run/libvirt/qemu/vm-name.monitor
```

```
Solution:
```
# Simplify VM configuration
virt-install --name vm1 --memory 1024 --vcpus 1

# Disable problematic features
qemu-system-x86_64 -no-hpet -no-acpi

# Use different machine type
virt-install --name vm1 --machine pc

# Check disk image integrity
qemu-img check disk.qcow2

# Enable serial console
qemu-system-x86_64 -serial pty -monitor stdio
```

**2. Performance Issues**

```
Symptoms:
- VM runs slowly
- High CPU usage on host
- Poor I/O performance

Diagnosis:
# Monitor host CPU
top -H -p qemu-system-x86_64

# Check KVM configuration
lsmod | grep kvm

# Monitor VM internals
virsh qemu-monitor-command vm-name "info status"

# Check disk I/O
iotop -o -a

# Check network I/O
iftop -i br0
```

```
Solution:
```
# Enable KVM
qemu-system-x86_64 -enable-kvm

# Use virtio drivers
virt-install --disk bus=virtio --network model=virtio

# Enable virtio multiqueue
virt-install --cpu host-passthrough,cache=writeback

# Tune scheduler
echo 1 > /sys/module/kvm/parameters/lapic
```

**3. Memory Issues**

```
Symptoms:
- Out of memory errors
- Swap usage high
- Memory leak in QEMU process

Diagnosis:
# Check host memory
free -h

# Check VM memory
virsh dommemstat vm-name

# Monitor KVM memory
cat /sys/module/kvm/parameters/hugepages

# Check QEMU memory
ps -o pid,vsz,pmem -C qemu-system-x86_64

# Check for ballooning
virsh dominfo vm-name | grep balloon
```

```
Solution:
```
# Enable memory ballooning
virt-install --balloon virtio

# Enable hugepages
echo 1 > /sys/kernel/mm/hugepages/hugepages-2048kB
echo 1024 > /proc/sys/vm/nr_hugepages

# Tune memory allocation
virt-install --memory 1024 --memballoc=interleave

# Limit VM memory
virsh setmem vm-name 512M

# Disable unused features
virt-install --no-usb --no-sound
```

#### Ceph Issues

**1. OSD Performance Problems**

```
Symptoms:
- Slow I/O operations
- High latency
- OSD rebalancing constantly

Diagnosis:
ceph tell osd.* iostat
ceph osd perf

# Check OSD stats
ceph osd dump_ops osd.0

# Check PG states
ceph pg dump

# Monitor recovery
ceph -w
```

```
Solution:
```
# Tune OSD configuration
ceph config set osd osd_op_threads 2
ceph config set osd osd_max_backfills 1
ceph config set osd osd_recovery_max_active 3

# Enable BlueStore
ceph config set osd osd_objectstore bluestore

# Tune memory
ceph config set osd osd_memory_target 4G

# Adjust max open files
ceph config set osd max_open_files 4096
```

**2. PG Distribution Issues**

```
Symptoms:
- CRUSH misplacement
- Uneven data distribution
- High latency on specific OSDs

Diagnosis:
ceph osd tree
ceph pg dump

# Check CRUSH map
ceph osd getcrushmap -o crush.map

# Analyze PG distribution
ceph pg dump | grep pg_num

# Check OSD utilization
ceph osd perf
```

```
Solution:
```
# Adjust PG count
ceph osd pool set <pool-name> pg_num <new-pg-count>

# Rebalance cluster
ceph osd reweight-by-utilization

# Check CRUSH rules
ceph osd getcrushmap -o crush.map
crushtool -i crush.map --test --num-rep 3 --rules <rule-id>

# Adjust CRUSH tunables
ceph config set osd osd_max_backfills 1
ceph config set osd osd_recovery_max_active 3
```

**3. Network Partition Issues**

```
Symptoms:
- OSDs marked down
- Network timeouts
- Split-brain scenario

Diagnosis:
ceph -s
ceph quorum_status

# Check network connectivity
ping -c 3 <mon-host>

# Monitor OSD status
ceph osd tree

# Check Corosync status
corosync-cfgtool -s
```

```
Solution:
```
# Adjust network timeouts
totem {
    token: 10000           # Increase timeout
    join: 120
}

# Enable redundant network
totem {
    interface {
        ringnumber: 1
        bindnetaddr: 192.168.2.10
        mcastport: 5405
    }
}

# Check firewall
iptables -L -n corosync

# Use dedicated network
# Separate cluster and public networks
```

### Performance Analysis

#### Profiling Techniques

**Corosync:**
```bash
# Profile with perf
perf record -e cycles,instructions -g -o perf.data corosync

# Analyze token circulation
perf report -i perf.data -s token

# Profile memory usage
perf record -e cycles -g -o mem.data corosync
```

**Pacemaker:**
```bash
# Profile PE calculations
time crm_simulate --simulate > /dev/null

# Monitor PE frequency
grep "calculated transition" /var/log/pacemaker/pengine/*
```

**QEMU:**
```bash
# Profile QEMU
perf record -e cycles,instructions,cache-misses -g -o qemu.perf \
    qemu-system-x86_64 ...

# Profile KVM operations
perf record -e cycles,instructions,kvm:kvm_exit,kvm:kvm_entry -g -o kvm.perf \
    qemu-system-x86_64 -enable-kvm ...
```

**Ceph:**
```bash
# Profile OSD
perf record -e cycles,instructions,cache-misses -g -o osd.perf \
    -p <osd-pid>

# Profile network
perf record -e cycles,instructions,kvm:kvm_exit -g -o network.perf \
    tcpdump -i any 'port 6800'

# Profile I/O
iostat -x 5 -d -c <rados-device>
```

#### Memory Analysis

**Detecting Memory Leaks:**
```bash
# Monitor process memory
watch -n 1 'ps -o pid,vsz,pmem -C corosync'

# Check for memory growth
ps -o pid,rss,vsz -C pacemaker \
    | awk '{print $2}' | sort -n

# Check for unfreed objects
valgrind --leak-check=full ./corosync

# Analyze heap usage
gdb -p corosync -batch -ex "bt"
```

#### Concurrency Issues

**Pacemaker:**
```bash
# Monitor concurrent operations
crm_mon --show-detail

# Check for deadlocks
grep "deadlock" /var/log/pacemaker/*

# Monitor transition queue
crm_simulate --show-failcounts
```

**Ceph:**
```bash
# Check for lock contention
ceph daemon --admin socket tell osd.0 dump_historic_ops

# Monitor thread pool
ceph tell osd.0 dump_ops_in_flight

# Check recovery activity
ceph -w
```

### Log Analysis

#### Understanding Log Levels

**Corosync:**
- `DEBUG`: Detailed diagnostic information
- `INFO`: General informational messages
- `WARN`: Warning conditions
- `ERROR`: Error conditions

**Pacemaker:**
- `notice`: Information about cluster events
- `warning`: Potential issues
- `error`: Error conditions
- `crit`: Critical failures

**Ceph:**
- `debug/10`: Detailed information (level 10)
- `debug/20`: More information
- `info`: General messages
- `warn`: Warning messages
- `err`: Error messages

#### Log Locations

```
Corosync:
/var/log/corosync/corosync.log
/var/log/corosync/debug.log
/var/log/syslog

Pacemaker:
/var/log/pacemaker/pengine/*
/var/log/pacemaker/crmd/*
/var/log/pacemaker/attrd/*

Ceph:
/var/log/ceph/ceph.log
/var/log/ceph/ceph-mon.log
/var/log/ceph/ceph-osd.*.log
```

#### Log Analysis Commands

**Corosync:**
```bash
# Token analysis
grep "token" /var/log/corosync/corosync.log

# Quorum events
grep "quorum" /var/log/corosync/corosync.log

# Network issues
grep "network" /var/log/corosync/corosync.log
```

**Pacemaker:**
```bash
# Find failed resources
grep "FAILED" /var/log/pacemaker/pengine/*

# Analyze transitions
grep "calculated transition" /var/log/pacemaker/pengine/*

# Check for errors
grep "error" /var/log/pacemaker/*
```

**Ceph:**
```bash
# OSD crashes
grep "segfault" /var/log/ceph/ceph-osd.*.log

# Slow requests
grep "slow request" /var/log/ceph/ceph-osd.*.log

# Network issues
grep "timed out" /var/log/ceph/ceph-osd.*.log
```

### Configuration Problems

#### Validation and Testing

**Before Applying Changes:**
```bash
# Validate Pacemaker configuration
crm_verify -L -V

# Test with simulation
crm_simulate --live-check

# Dry-run Ceph commands
ceph tell osd.* injectargs --dry-run

# Test iSCSI configuration
gwcli.py target list
```

**Rollback Procedures:**
```bash
# Pacemaker snapshot
cibadmin --query --local > cib-backup.xml

# Rollback if needed
cibadmin --replace --local --xml-file cib-backup.xml

# Restore Ceph configuration
ceph config set <option> <previous-value>
```

### Integration Issues

#### Cluster Coordination

**Corosync and Pacemaker:**
```bash
# Check Corosync status from Pacemaker
crm_resource --list

# Verify quorum
crm_attribute -q -n quorum

# Check node status
pcs status nodes

# Ensure Pacemaker can communicate
crm_mon --show-detail
```

**Ceph with Cluster Managers:**
```bash
# Check if cluster manager sees Ceph
ceph -s

# Verify network connectivity
ping <mon-host>

# Check monitor status
ceph -w
```

#### Storage Integration

**iSCSI Issues:**
```bash
# Check gateway status
gwcli.py target list

# Check initiator connections
iscsiadm -m session

# Check multipath status
multipath -ll

# Reset initiator state
iscsiadm -m node logout -T <target-iqn> -p <portal-ip>
iscsiadm -m node login -T <target-iqn> -p <portal-ip> --login
```

**RBD Issues:**
```bash
# Check mapped images
rbd showmapped

# Check image status
rbd info <pool>/<image>

# Unmap and remap
rbd unmap <pool>/<image>
rbd map <pool>/<image>

# Check for stale mappings
dmesg | grep rbd
```

### Advanced Diagnostics

#### System-Level Debugging

```bash
# Enable kernel debugging
echo 1 > /proc/sys/kernel/sched_sched_min_granularity
echo 1 > /proc/sys/kernel/khung_task_timeout_secs

# Enable KVM tracing
echo 1 > /sys/module/kvm/parameters/trace_gue

# Enable Corosync tracing
corosync-cmapctl totem.trace_buffer_size 1000000

# Monitor system resources
top -b -H -d 2
vmstat 1
iostat -x 2
```

#### Network Diagnostics

```bash
# Check network latency
ping -i 3 -s 64 <target-host>

# Trace routes
traceroute -n <target-host>

# Check bandwidth
iperf -c 3 -t 10 <target-host>

# Capture packets
tcpdump -i any -w /tmp/capture.pcap host <target-ip>
```

### Best Practices for Troubleshooting

1. **Always start with debug enabled** when investigating issues
2. **Collect logs before clearing them** - keep a complete copy
3. **Document the issue** - note exact error messages, timestamps
4. **Test in isolated environment** if possible
5. **Check configuration drift** - ensure all nodes have same configuration
6. **Monitor system resources** - CPU, memory, disk I/O, network
7. **Use incremental debugging** - narrow down the problem step by step
8. **Have rollback plan** - know how to quickly revert changes

### Additional Resources

- **Corosync Wiki**: https://github.com/corosync/corosync/wiki/Troubleshooting
- **Pacemaker Guide**: https://www.clusterlabs.org/pacemaker/doc/2.0/Pacemaker_Explained/
- **Ceph Troubleshooting**: https://docs.ceph.com/en/latest/rados/troubleshooting/
- **QEMU Debugging**: https://wiki.qemu.org/Documentation/Debugging/
- **Linux Performance**: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/tuning_guide/
