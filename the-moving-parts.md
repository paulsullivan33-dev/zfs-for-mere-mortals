# The Moving Parts

Six ideas carry the whole of ZFS. Learn
these and every command in the rest of
the guide is a variation on a theme.

## 1. The pool: one bucket of storage

A **pool** (you will also see its old
name, *zpool*, in every command) is all
the storage ZFS manages for you, made
from one or more disks. Files are stored
in the pool, not on a disk: you stop
thinking "the big disk" and "the spare
disk" and start thinking "the pool has
room" or "the pool is full".

A healthy home pool is usually called
something short and lowercase — `tank`
is the traditional example name, and
this guide uses `tank` for real pools
and `practice` for the throwaway pool
made of files.

## 2. Vdevs: how the disks combine

Disks join a pool grouped into **virtual
devices**, or *vdevs*. The vdev is where
redundancy lives. The shapes that matter
at home:

| Vdev shape | Disks | Survives losing | Cost |
|---|---|---|---|
| **Single** | 1 | Nothing | None |
| **Mirror** | 2 or more | All but one disk in the mirror | Half the space (with 2) |
| **RAIDZ1** | 3 or more | Any 1 disk | One disk's worth of space |
| **RAIDZ2** | 4 or more | Any 2 disks | Two disks' worth |
| **RAIDZ3** | 5 or more | Any 3 disks | Three disks' worth |

A pool stripes data across its vdevs,
and the pool survives only what *every*
vdev survives: a pool of a mirror *plus*
a single loose disk dies when the loose
disk does. That one sentence is the most
expensive misunderstanding in ZFS, and
chapter 4 returns to it.

Two vdevs hold no files at all. A **log**
vdev (called a SLOG) only absorbs
synchronous writes quickly, and a
**cache** vdev (an L2ARC) only holds a
second-level read cache. Both are
optimisations for workloads most homes
don't have — this guide names them so
you recognise the terms, and otherwise
leaves them alone.

## 3. Datasets: folders with settings

Inside a pool you make **datasets** —
one per job: `tank/photos`,
`tank/media`, `tank/backups`. A dataset
looks like a folder and mounts like one,
but it carries its own settings, called
**properties**: whether its files are
compressed, whether access times are
recorded, a size limit, and more.
Properties are *inherited*, so setting
compression on `tank` covers every
dataset under it unless one overrides it.
Chapter 5 lives here.

There are also **volumes** (*zvols*) —
datasets that appear as raw block
devices, used for virtual machines and
iSCSI. This guide stays with ordinary
file datasets.

## 4. Checksums and the block pointer tree

Every file is a tree of blocks. The
metadata that points to each block also
holds that block's checksum, and the
metadata is itself checksummed by the
level above it, up to a root block that
anchors the whole dataset. Reading a
file re-verifies the whole chain.
Corruption anywhere is found at the
exact block that went bad — which is why
`zpool status` can name the affected
file, and why a **scrub** (chapter 8)
can audit an entire pool by simply
reading everything, carefully.

## 5. Copy-on-write and transactions

Writes go to fresh space; the pointers
flip to the new version in one atomic
step, grouped as a *transaction*. Crash
at any moment and the pool is at some
whole transaction boundary — consistent,
mountable, no checking pass needed. The
flip side: a nearly full pool struggles
to find fresh space, so ZFS pools should
keep breathing room. The working rule is
to plan an alert at about 80% full —
not a law of physics, but the point
where copy-on-write gets cramped and
performance falls away.

## 6. The ARC: memory is the cache

ZFS caches aggressively in RAM, in the
**Adaptive Replacement Cache** (ARC).
Frequently and recently used data is
served from memory, which is why a ZFS
server feels faster the longer it runs —
and why ZFS has a reputation for eating
RAM. By design, the ARC grows into
available memory and gives it back when
other programs need it, though not
always as politely as you'd like on a
small machine. Chapter 12 covers sizing
honestly; the short version is that ZFS
works on modest memory and is *happier*
with more, and that one feature —
deduplication — is the exception that
really does demand RAM by the bucket.
This guide tells you to leave dedup off.

## Check: the six parts, named

Say which part each line names:

```bash
zpool status          # the pool and its vdevs
zfs list              # the datasets
zpool get all tank    # pool properties
zfs get compression tank/photos
                      # a dataset property, inherited or set
arc_summary           # the cache (a separate tool, where installed)
```

If you can place pool, vdev, dataset,
checksum, copy-on-write and cache,
chapter 3 can put real software under
them.
