# ZFS for Mere Mortals

Every disk you own is slowly, quietly
failing. Most filesystems cannot tell —
they hand you the bytes the disk gave
them and let your backups copy the
damage faithfully. **ZFS** is the
filesystem and volume manager built to
notice: it checksums every block it
stores, catches corruption at the moment
it is read, repairs it from a second
copy when the pool has one, and throws
in snapshots, clones and built-in
replication as consequences of the same
design. This guide explains how it
works, how to plan a pool before you
build one, and how to run it for years
without drama: datasets tuned to their
jobs, time travel for your mistakes,
scrubs that audit everything monthly,
and backups that are actually a
different pool in a different place.

The lab behind this guide is stated
plainly, because its ceiling matters:
the sandbox that built this guide ran
the OpenZFS userspace tools but **could
not load the ZFS kernel module**, so no
pool could be created in it. (The
straight package install was also
blocked — by an unrelated, half-finished
package change elsewhere in the shared
sandbox — so the lab fetched the
distribution's exact `zfsutils-linux`
2.2.2 packages and ran their genuine
binaries unpacked, rather than claiming
an install that never completed.) What
was verified live was the userspace
itself — the versions of `zpool` and
`zfs` Ubuntu 24.04 currently ships, their
complete help surfaces, and the precise
refusal the tools give when the module
is missing. The
pool, dataset, snapshot and replication
chapters are therefore written from the
OpenZFS documentation and the projects'
own manuals, and are labelled
**documented-only** wherever a claim
rests on reading rather than running.
That is the same honesty pattern this
series used for Kubernetes and
Terraform: say what was run, say what
was only read, and never let the reader
mistake one for the other. The full
ledger is in `criteria.md`.

## What this guide covers

- [**What ZFS is**](what-zfs-is.md) —
  a filesystem and volume manager in
  one, checksumming everything, healing
  what it can, and what it is not
- [**The moving parts**](the-moving-parts.md) —
  pools, vdevs, datasets, the checksum
  tree, copy-on-write, and the ARC cache:
  the six ideas everything else uses
- [**Installing ZFS**](installing-zfs.md) —
  the Linux route the lab performed, the
  DKMS and Secure Boot traps, and why a
  sandbox that cannot load kernel modules
  cannot run ZFS at all
- [**Your first pool**](your-first-pool.md) —
  a practice pool made of files where
  mistakes are free, then the three
  decisions a real pool demands
- [**Datasets and properties**](datasets-and-properties.md) —
  folders with settings: inheritance,
  compression, quotas, and the one
  property to leave off
- [**Snapshots, clones and rollback**](snapshots-clones-and-rollback.md) —
  time travel for a dataset, looking
  inside a snapshot, and what snapshots
  really cost
- [**Sending and receiving**](sending-and-receiving.md) —
  replication built in: full sends,
  incremental sends, bookmarks, and the
  raw encrypted stream
- [**Scrubs, repair and replacing a disk**](scrubs-repair-and-replacing-a-disk.md) —
  the monthly audit, reading error
  counters, and the calm procedure for
  the day a disk dies
- [**Watching the pool**](watching-the-pool.md) —
  the five-minute monthly routine, pool
  events and ZED, and the space
  accounting that answers "where did it
  all go?"
- [**Backup and recovery**](backup-and-recovery.md) —
  why a pool is not a backup, replication
  as the backup mechanism, native
  encryption, and the recoveries
  rehearsed in advance
- [**Recipes**](recipes.md) — the
  self-healing photo archive, the
  growing media shelf, the backups
  landing zone, the VM shelf, and the
  family share with ZFS underneath
- [**The neighbours**](the-neighbours.md) —
  ext4, XFS, Btrfs, the LVM and mdadm
  stack, SnapRAID with MergerFS, and
  TrueNAS: what each one trades
- [**Limits**](limits.md) — the shape
  chosen at creation, full pools, memory,
  no clustering, and the licence that
  shaped ZFS's Linux life
- [**Troubleshooting**](troubleshooting.md) —
  modules that won't load, pools that
  vanish until import, space that
  snapshots are holding, and errors that
  are cables, not disks
- [**Glossary**](glossary.md) — the
  terms, collected in one place

## What it doesn't cover

Root-on-ZFS installs (booting the whole
system from a pool is its own art, and
your distribution's installer has
opinions), ZFS on macOS or Windows,
tuning for databases and other
specialist workloads beyond the property
guidance in chapter 5, and multi-host
high availability — chapter 12 explains
why that last one was never ZFS's job.

## How to use this guide

Read chapters 1 and 2 before touching a
disk — knowing that redundancy lives in
vdevs and that settings inherit turns
later confusion into named problems.
Chapter 3 installs; chapter 4 builds
the *practice* pool first, and you
should do chapters 5 to 8's commands
there before any real disk is involved.
Chapter 10 is not the optional ending:
a pool without replication and a
rehearsed restore is a single point of
failure with excellent checksums. The
lab's transcripts live in
`zfs-lab/transcripts/`; its notes are
`examples/lab-notes.txt`, and the pool
plan worth copying is
`examples/sample-pool-plan.txt`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [SnapRAID for Mere Mortals](https://paulsullivan33-dev.github.io/snapraid-for-mere-mortals/)
- [Backups for Mere Mortals](https://paulsullivan33-dev.github.io/backups-for-mere-mortals/)
- [Proxmox for Mere Mortals](https://paulsullivan33-dev.github.io/proxmox-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/zfs-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/zfs-for-mere-mortals/blob/main/LICENSE-CODE).
