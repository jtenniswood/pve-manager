# Proxmox VE interface feature preservation matrix

Preserve the existing resource hierarchy and every management workflow as the interface changes. This matrix groups the initial audit scope; it is not a completed behavior audit. All areas remain **planned** for regression review. The recommended approach refreshes the existing ExtJS interface; this matrix also provides a baseline if a future framework migration is agreed.

## Functional areas

Paths below are relative to `www/manager6/`, except shared toolkit references.

| Area | Source anchors | Behaviors to audit and retain |
| --- | --- | --- |
| Workspace and resource tree | `Workspace.js`, `tree/`, `data/`, `panel/ConfigPanel.js`, `StateProvider.js`, `UIOptions.js` | Datacenter/node/guest/storage/pool/SDN selection, alternate tree views, tags, nested pools, search, sorting, context menus, bulk actions, saved state and old links |
| Login and user preferences | `window/LoginWindow.js`, `window/TFALoginWindow.js`, `window/Settings.js`, `Workspace.js` | Realms, consent, language/theme, password/TFA/OpenID flows, renewal, logout, user settings, subscription messaging |
| Datacenter overview and settings | `dc/Config.js`, `dc/Summary.js`, `dc/OptionView.js` | Summary, notes, cluster membership/join, cluster Ceph, global options, support |
| Identity and permissions | `dc/`, `panel/`, shared toolkit | Users, groups, roles, realms, ACLs and inheritance, API tokens, TFA setup and recovery, password operations |
| Cluster and HA | `dc/Cluster.js`, `ha/`, `window/BulkAction.js` | Cluster operations, HA resources/rules/fencing, state and placement, cluster-wide guest operations |
| Backup scheduling and notifications | `dc/Backup*.js`, `dc/NotificationEvents.js`, shared toolkit notification views | Jobs, schedules, selection/exclusions, retention, notification targets/matchers and settings |
| Datacenter firewall | `dc/Config.js`, `panel/Firewall*.js`, `grid/Firewall*.js` | Ordered rules, enablement, options, aliases, IP sets, security groups, logs and capabilities |
| SDN | `sdn/` | Zones, VNets/subnets, controllers, IPAM, DNS, mappings, firewall, fabrics and protocol-specific editors, route maps, prefix lists, status and pending apply |
| Mappings and CPU models | `dc/*Map*.js`, `dc/CPUType*.js`, `form/` | PCI, USB and directory mappings, node-specific definitions, custom CPU models |
| Metrics and certificates | `dc/MetricServerView.js`, `node/Certificates.js`, ACME-related windows, shared toolkit | Metric servers, ACME accounts/plugins, node certificate upload/renewal and task results |
| Node overview and actions | `node/Config.js`, `node/Summary.js` | Summary, notes, power actions, bulk guest operations, shell, task history, subscription |
| Node system configuration | `node/Config.js`, shared `node/` views | Services, bridges/bonds/VLANs, DNS, hosts, time, node options, journal, package updates and repositories |
| Node disks | `node/`, shared disk views | Inventory, SMART, wipe/init, directory creation, LVM/LVM-thin, ZFS and associated destructive confirmations |
| Node firewall and replication | `node/Config.js`, `grid/Replication.js`, firewall views | Node rules/options/logs, replication schedules/status/actions |
| Ceph | `ceph/`, `node/Config.js`, Ceph windows | Installation/configuration, status, monitors/managers, OSDs, pools, CephFS/MDS, CRUSH, logs and recovery operations |
| VM creation and hardware | `qemu/CreateWizard.js`, `qemu/HardwareView.js`, `qemu/*Edit.js`, `form/` | Every wizard field; CPU/memory/disks/network; boot, firmware, TPM, EFI, PCI/USB, audio/display and advanced settings |
| VM configuration and runtime | `qemu/Config.js`, `qemu/Options.js`, `qemu/Ci*.js`, `qemu/Monitor.js` | Options, cloud-init, agent information, pending/hotplug settings, monitor, lifecycle, protection, templates |
| Containers | `lxc/`, `container/`, `form/` | Creation/templates, resources/mount points, networking/DNS, features, privilege mode, options, console, lifecycle and lock states |
| Guest mobility and duplication | `window/Migrate.js`, `window/Clone.js`, `window/GuestImport.js`, guest menus | Online/offline/remote migration, disk/storage selection, clone/template conversion, import mappings and progress |
| Guest backup and restore | `panel/Backup*.js`, `window/Backup.js`, `window/Restore.js`, storage backup views | Backup execution, restore, retention/protection, archive browsing and downloads where supported |
| Snapshots and replication | `tree/SnapshotTree.js`, `grid/Replication.js`, guest configs | Creation/deletion/rollback, metadata, snapshot hierarchy, RAM-state choices, replication settings/actions |
| Guest firewall and ACLs | `qemu/Config.js`, `lxc/Config.js`, firewall/ACL views | Rules, options, aliases, IP sets, logs, guest-specific capabilities and permissions |
| Storage definitions | `dc/StorageView.js`, `storage/*Edit.js`, `storage/Base.js` | All currently exposed backend editors: directory, LVM/thin, NFS, CIFS, iSCSI, ZFS/ZFS pool, RBD, CephFS, PBS, BTRFS, ESXi; supported fields and constraints |
| Storage content | `storage/Browser.js`, `storage/*View.js`, upload/download windows | Backups, VM disks, CT volumes, ISO images, CT templates, snippets, imports; upload, URL download, allocation, move/delete and permissions |
| Pools and SDN resources | `pool/`, `sdn/Browser.js` | Pool summary/members/ACLs, nested membership, SDN content and VRF views |
| Tasks and consoles | `Utils.js`, `panel/`, console buttons; shared task views | Global/node/guest task history, log pagination, cancellation, noVNC, xterm.js, SPICE and supported console preferences |

File globs identify audit families, not an assertion that every named operation lives in a single file. Resolve the complete implementation and API path before porting it. The machine-readable inventory records exact existing paths and navigation anchors.

## Workflow record template

Create one record for each meaningful operation and its variants:

| Field | Required content |
| --- | --- |
| Identity | Resource type, existing label, old route/tab, affected presentation or proposed route |
| Source | All relevant PVE and shared toolkit files, pinned revision |
| Availability | Privileges, guest state, template/lock flags, cluster/storage prerequisites |
| API contract | Endpoint, method, request encoding, parameters, defaults, delete/digest rules, response/error format |
| Interaction | Fields, advanced options, validation, help, confirmations, keyboard behavior |
| Outcome | Synchronous result or UPID task; active/pending config; refreshed affected resources |
| Failure paths | Denied access, stale config, timeout, offline node/storage, partial or failed task |
| Evidence | API/interaction/browser/lab scenarios and actual results |
| Status | Planned, audited, changed, or verified; record unchanged behavior explicitly |
| Review | Responsible contributor, reviewing maintainer, upstream issue/patch/PR link |

## Release gates

A refreshed area is complete only when its affected workflows and conditional variants are verified, including permission-denial and failure paths. Visual similarity and API success alone are insufficient.

The default appearance can change only after the affected components and functional areas have been audited, required workflows remain usable, old links and preferences still work, packaged upgrades and rollback pass, and upstream agrees on the rollout. For a future framework replacement, every required workflow must additionally be implemented and verified in that new interface. Continue updating the audit for features added upstream during the work.
