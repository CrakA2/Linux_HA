# Storage and File Systems

## [CEPH](ceph.md)
Distributed storage system providing object, block, and file storage in unified platform.

## [iSCSI](iscsi.md)
iSCSI gateway presenting RBD images as SCSI disks over TCP/IP network.

## [NVMe-oF](nvme-of.md)
NVMe over Fabrics gateway providing high-performance block access to Ceph.

## [GFS2](gfs2.md)
Global File System 2 for shared-disk file system in Linux clusters.

## CEPH HCI Deployment

```bash
# 1. Install cephadm
curl --silent --remote-name --location \
    https://download.ceph.com/rpm-18.2.1/el9/noarch/cephadm \
    -o cephadm
chmod +x cephadm

# 2. Bootstrap cluster
./cephadm bootstrap --mon-ip <mon-ip>

# 3. Add storage
ceph orch apply osd --all-available-devices

# 4. Verify cluster
ceph -s
ceph status
```

## Security

- Enable CephX encryption
- Use CHAP authentication for iSCSI
- Use RDMA/RoCE for secure transport
