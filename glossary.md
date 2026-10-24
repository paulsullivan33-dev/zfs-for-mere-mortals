# Glossary

**ARC (Adaptive Replacement Cache)** —
ZFS's read cache in RAM: recent *and*
frequent data, served from memory.
Grows into free RAM, shrinks under
pressure. Chapter 2.

**ashift** — the pool's record of the
disks' sector size, set at creation
(12 = 4K sectors) and unchangeable
after. Wrong guesses cost performance
permanently. Chapter 4.

**Bookmark** — a marker standing in for
a snapshot: keeps the position an
incremental send needs, without pinning
the snapshot's data. Written with `#`
where a snapshot uses `@`. Chapter 7.

**Checksum** — a fingerprint of a block,
stored away from the block, verified on
every read. The heart of ZFS. Chapters
1 and 2.

**Clone** — a writable dataset created
from a snapshot; shares its blocks until
they diverge. Chapter 6.

**Copy-on-write** — writes go to fresh
space and pointers flip atomically; old
versions remain until freed. Crash-safe
by construction, and the reason
snapshots are cheap. Chapter 2.

**Dataset** — a named, separately
managed slice of a pool, with its own
properties, snapshots and quota. Looks
like a folder, acts like a filesystem.
Chapter 5.

**Dedup (deduplication)** — storing
identical blocks once, pool-wide. RAM-
hungry; this guide advises leaving it
off. Chapter 5.

**L2ARC** — a second-level read cache on
a fast device (usually an SSD), fed by
the ARC. A "cache vdev". Chapter 2.

**Mirror** — a vdev whose disks each
hold everything; survives all but one
of them failing. Chapter 2.

**Pool (zpool)** — all the storage ZFS
manages together, built from vdevs,
divided into datasets, referred to by a
name like `tank`. Chapter 2.

**Property** — a named setting on a
pool or dataset (`compression`, `quota`,
...). Inherited from parents unless set
locally. Chapter 5.

**RAIDZ1 / 2 / 3** — parity vdevs:
data plus one, two or three disks'
worth of parity, surviving that many
disk failures. Chapter 2.

**Recordsize** — the largest block a
dataset writes in one piece; a property
to match to the workload. Chapter 5.

**Resilver** — rebuilding a replaced
disk from the surviving copies: only
live data is copied. Chapter 8.

**Rollback** — rewinding a dataset to a
snapshot, discarding everything written
after it. Chapter 6.

**Scrub** — a full read-and-verify of
the pool, repairing from redundancy;
run on a schedule. Chapter 8.

**Send / receive** — writing a snapshot
to a stream, and rebuilding a dataset
from such a stream: ZFS's replication.
Chapter 7.

**SLOG** — a separate log device (a
"log vdev") absorbing synchronous
writes. An optimisation most homes
don't need. Chapter 2.

**Snapshot** — a read-only, point-in-
time view of a dataset, kept by
copy-on-write at near-zero cost until
the data diverges from it. Chapter 6.

**Vdev (virtual device)** — a group of
disks in one shape — mirror, RAIDZ,
single — joined to a pool. Redundancy
lives at this level. Chapter 2.

**ZED** — the ZFS Event Daemon: watches
pool events and can run scripts or send
mail when the pool needs a human.
Chapter 9.

**Zvol** — a dataset presented as a raw
block device, for VM disks and iSCSI.
Named in chapter 2; this guide stays
with file datasets.
