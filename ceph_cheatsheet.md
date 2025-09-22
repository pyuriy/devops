# Ceph Comprehensive Cheat Sheet

Ceph is an open-source distributed storage system that provides object, block, and file storage. This cheat sheet compiles key commands, configurations, and best practices for managing a Ceph cluster, drawn from official documentation and community resources. It covers setup, monitoring, and service-specific management. For production, always refer to the latest Ceph documentation (e.g., Pacific or later releases as of 2025).

## Components Overview
- **OSD (Object Storage Daemon)**: Stores data on individual drives.
- **MON (Monitor)**: Maintains cluster map; use odd number (3 or 5 recommended).
- **MGR (Manager)**: Handles statistics, dashboard, and modules like balancer.
- **MDS (Metadata Server)**: Manages CephFS metadata.
- **Pools**: Logical partitions for data with redundancy settings.
- **PGs (Placement Groups)**: Subdivisions of pools for data distribution.

**Hardware Recommendations**:
- OSD: 1-4 GiB RAM per drive, fast CPU.
- MON/MGR: Fast SSD storage.
- Network: 10Gbps+ recommended.

## Setup and Installation
### Basic Cluster Bootstrap (Using cephadm)
- Bootstrap: `cephadm bootstrap --mon-ip <IP> --allow-fqdn-hostname`
- Add host: `ceph orch host add <hostname> <IP>`
- Deploy MONs: `ceph orch apply mon --placement="3 <host-pattern>"`
- Deploy MGRs: `ceph orch apply mgr --placement="1 each"`
- Enable dashboard: `ceph mgr module enable dashboard; ceph dashboard create-self-signed-cert`

### Single-Node Testing (Not for Production)
- `ceph config set global osd_pool_default_size 1`
- `ceph config set global mon_allow_pool_size_one true`
- `ceph orch apply mon --placement=1`
- `ceph orch apply mgr --placement=1`

### Adding OSDs
| Command | Description |
|---------|-------------|
| `ceph orch device ls` | List available devices for OSDs. |
| `ceph orch daemon add osd <host>:/dev/<disk>` | Add OSD on specific device. |
| `ceph orch apply osd --all-available-devices` | Deploy OSDs on all eligible devices. |
| `ceph osd set noout` | Prevent data movement before adding OSDs (unset with `ceph osd unset noout`). |
| `ceph osd in <id>` | Mark OSD as in after adding. |

**LVM Setup for OSD**: `pvcreate /dev/<disk>; vgcreate ceph-vg /dev/<disk>; ceph orch daemon add osd <host>:ceph-vg`

### Monitor Configuration
- Extract monmap: `ceph-mon --extract-monmap /tmp/monmap`
- Add MON: Edit monmap with `monmaptool`, then `ceph-mon --inject-monmap /tmp/monmap`
- Set min OSD ratio: `ceph config set mon mon_osd_min_in_ratio 0.8`

### Manager Setup
- Create key: `ceph auth get-or-create mgr.<id> mon 'allow profile mgr' osd 'allow *' mgr 'allow *'`
- Enable: `systemctl enable --now ceph-mgr@<id>`
- Enable modules: `ceph mgr module enable dashboard; ceph mgr module enable balancer`

## Cluster Monitoring and Health
| Command | Description |
|---------|-------------|
| `ceph -s` | Cluster status summary (health, OSDs, PGs). |
| `ceph -w` | Watch cluster status in real-time. |
| `ceph health detail` | Detailed health warnings/errors. |
| `ceph df` | Global and per-pool usage. |
| `rados df` | Detailed pool object stats. |
| `ceph osd df tree` | OSD usage with CRUSH hierarchy. |
| `ceph osd tree` | OSD hierarchy, status, weights. |
| `ceph osd stat` | OSD map summary (up/in count). |
| `ceph config dump` | All cluster configurations. |

**Performance Counters**: `ceph daemon <osd.id> perf dump`

## Pool Management
| Command | Description |
|---------|-------------|
| `ceph osd pool create <name> <pg_num>` | Create replicated pool (e.g., 128 PGs). |
| `ceph osd pool ls [detail]` | List pools with details. |
| `ceph osd pool get <name> all` | Get all pool parameters. |
| `ceph osd pool set <name> size <val>` | Set replication size (e.g., 3). |
| `ceph osd pool set <name> pg_num <val>` | Set PG count (power of 2 recommended). |
| `ceph osd pool delete <name> <name> --yes-i-really-really-mean-it` | Delete pool (confirm twice). |

**For Testing (Size 1)**: `ceph osd pool set <pool> size 1 --yes-i-really-mean-it`

## OSD Management
| Command | Description |
|---------|-------------|
| `ceph osd out <id>` | Mark OSD out (triggers rebalance). |
| `ceph osd in <id>` | Mark OSD in. |
| `ceph osd reweight <id> <weight>` | Temporary weight override (0-1). |
| `ceph osd crush reweight <id> <weight>` | Permanent CRUSH weight. |
| `ceph osd reweight-by-utilization <percent>` | Auto-reweight based on usage. |
| `ceph osd scrub <id>` | Initiate regular scrub. |
| `ceph osd deep-scrub <id>` | Deep consistency check. |
| `ceph osd find <id>` | Locate OSD (host/port). |
| `ceph osd metadata <id>` | OSD metadata (host info). |
| `ceph config set osd osd_memory_target <bytes>` | Set OSD RAM target (e.g., 4GiB = 4294967296). |

**Remove OSD**: `ceph osd purge <id> --yes-i-really-really-mean-it`

## Placement Group (PG) Management
| Command | Description |
|---------|-------------|
| `ceph pg dump` | All PG stats. |
| `ceph pg dump_stuck degraded` | Stuck/degraded PGs. |
| `ceph pg <pgid> query` | PG details. |
| `ceph pg <pgid> list_missing` | Missing objects in PG. |
| `ceph pg <pgid> mark_unfound_lost delete` | Handle lost objects (delete/revert). |
| `ceph pg scrub <pgid>` | Scrub PG. |
| `ceph pg deep-scrub <pgid>` | Deep scrub PG. |
| `ceph pg repair <pgid>` | Repair inconsistent PG. |

**PG Calculation**: Aim for ~100 PGs per OSD; use `ceph osd pool get <pool> pg_num`.

## Daemon Interactions
**Subcommands of `ceph daemon <type>.<id>`**:
| Command | Description |
|---------|-------------|
| `ceph daemon mon.<id> mon_status` | MON status. |
| `ceph daemon osd.<id> status` | OSD status. |
| `ceph daemon osd.<id> dump_ops_in_flight` | Active operations. |
| `ceph daemon <type> help` | List supported commands. |

## CephFS Management
### Create and Deploy
- Pools: `ceph osd pool create cephfs_metadata 32; ceph osd pool create cephfs_data 128`
- Filesystem: `ceph fs new cephfs cephfs_metadata cephfs_data`
- MDS: `ceph orch apply mds cephfs 2` (2 for production)
- Status: `ceph fs ls; ceph mds stat`

**Client Mount**: `mount -t ceph <mon-ip>:6789:/ /mnt/cephfs -o name=admin,secret=<key>`

**Permissions**: `ceph auth caps client.admin mon 'allow *' mds 'allow *' osd 'allow *' mgr 'allow *'`

## RBD (Block Storage)
| Command | Description |
|---------|-------------|
| `rbd pool init <pool>` | Initialize RBD pool. |
| `rbd create <pool>/<image> --size <M>` | Create image (e.g., 10G). |
| `rbd ls -p <pool>` | List images in pool. |
| `rbd map <pool>/<image>` | Map image to block device. |
| `rbd unmap <device>` | Unmap device. |
| `rbd resize <pool>/<image> --size <M>` | Resize image. |
| `rbd snap create <pool>/<image>@<snap>` | Create snapshot. |

**Kernel Mount**: After mapping, `mkfs.ext4 /dev/rbd0; mount /dev/rbd0 /mnt`

## RGW (Object Storage)
- Deploy: `ceph orch apply rgw <realm>.<zone> --placement="2 <host>"`
- Create realm/zone: `radosgw-admin realm create --rgw-realm=<name>; radosgw-admin zonegroup create --rgw-zonegroup=<name>`
- User: `radosgw-admin user create --uid=<user> --display-name=<name>`
- S3 Access: Use AWS CLI with `endpointurl=http://<rgw-ip>`

**NFS Export via RGW** (In Progress in Sources): Create service `ceph orch apply nfs <cluster> --placement=1`

## NFS-Ganesha Integration
- Service: `ceph orch apply nfs exports --placement=1`
- Volume: `ceph fs volume create cephfs; ceph fs subvolumegroup create cephfs nfs`
- Export: `ceph nfs export create cephfs exports /volumes/nfs cephfs --path /volumes/nfs --pseudo-path /export1`
- Client Mount: `mount -t nfs <host>:/export1 /mnt`

## Authentication
| Command | Description |
|---------|-------------|
| `ceph auth get-key <entity>` | Get key for entity (e.g., client.admin). |
| `ceph auth list` | List all auth entities. |
| `ceph auth caps client.<id> <type> 'allow <perms>'` | Set capabilities (e.g., osd 'allow rwx'). |

## Troubleshooting
| Command | Description |
|---------|-------------|
| `ceph health detail` | Health issues. |
| `ceph -W mon` | Monitor warnings. |
| `ceph log last <daemon>` | Recent logs. |
| `ceph orch ps` | Daemon status. |
| `ceph orch logs --name <daemon>` | Daemon logs. |
| `ceph tell osd.* version` | Check versions. |
| `wipefs -a /dev/<disk>` | Wipe disk for reuse. |

**Common Flags**:
- `noout`: Prevent OSD out during maintenance.
- `norebalance`: Pause rebalancing.
- `norecover`: Pause recovery.

**Stuck PGs**: Query with `ceph pg <id> query`, repair if needed.

For advanced topics like iSCSI/SMB/S3 with RGW, consult Ceph docs as implementations evolve. Always test in non-production environments.
