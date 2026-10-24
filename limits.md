# Limits

What ZFS doesn't do, stated plainly
enough to plan around.

## The shape is chosen at creation

A pool's vdev layout is close to
permanent. You can *add* vdevs (of
matching protection, chapter 4's
sermon), and in recent OpenZFS versions
you can widen a RAIDZ vdev one disk at
a time or remove some top-level vdevs —
but you cannot turn a mirror into a
RAIDZ2, shrink a pool onto fewer disks,
or rebalance history. Data written to
old vdevs stays there; new writes
spread. Choose the shape for the pool
you'll want in three years, not the
disks you happen to have today, or
accept a rebuild-by-copying when the
shape changes (chapter 7 makes that
rebuild possible, and it is still a
rebuild).

## Full pools slow down, then stop being funny

Copy-on-write needs free space to write
into. A pool near full fragments badly
and write performance falls away; at
genuinely full, datasets that must grow
cannot, in ways that present as
applications failing. The community
guideline — plan around 80% — is in
chapter 2 because it is the limit you
will actually meet first. ZFS will not
warn you into safety here; chapter 9's
routine is the warning.

## Memory hunger is real (and mostly fair)

The ARC will use memory that is
otherwise idle, and on a general-purpose
machine doing other work, ZFS and your
applications can end up negotiating
rudely. You can cap the ARC
(`zfs_arc_max`) when you must. What you
cannot do is make a small-memory machine
perform like a big one: ZFS's speed
story *is* its cache. Deduplication is
the extreme case — its lookup table
grows with unique data and it can eat
memory until the machine swaps; the
guide's advice stands: leave dedup off
unless you have measured a workload
that begs for it.

## No in-place arrival, no shrinking datasets' past

There is no convert-in-place from ext4
or anything else; data arrives by
copying into a fresh pool. Likewise, a
dataset property changed today does not
rewrite yesterday's blocks — old files
keep the compression and recordsize
they were written with until copied
afresh. ZFS is a system that rewards
deciding *before* writing.

## It is not a cluster filesystem

A pool belongs to one machine at a time.
Two hosts may not import the same pool
(shared-disk setups force-import at
their peril — the warning ZFS gives
there is sincere), and ZFS offers
nothing like live multi-host access. If
several machines need the data, they
reach it over NFS or Samba like anything
else (chapter 11). High availability is
something you build *around* ZFS, not
something it includes.

## The licence shaped its Linux life

ZFS's CDDL licence is incompatible with
the Linux kernel's GPL, which is why
Linux gets ZFS as an add-on module
(OpenZFS) rather than built into the
kernel, why distributions vary in how
they ship it, and why FreeBSD — where
ZFS is a first-class citizen of the base
system — has the smoothest story.
Practically: on Linux, installing means
a package and a module (chapter 3), and
a kernel upgrade occasionally means
waiting for the module to match. This
is a known, managed irritation, not a
scandal — but it is why chapter 3's
checks exist.

## Detecting damage is not preventing it

A single-disk pool, or a `copies=1`
dataset, gets the alarm without the
repair (chapter 1). Also: ZFS verifies
data *it* stores. It cannot help with
the application that wrote the wrong
bytes beautifully checksummed, the RAM
error that corrupted data before ZFS
ever saw it (ECC memory reduces this;
ZFS neither requires ECC nor is harmed
by it — it wants exactly what every
filesystem wants, honest hardware), or
the administrator with root and a
typo. The last risk is the largest, and
chapter 6's snapshots plus chapter 10's
copies are its only real mitigation.

## Performance fine print

Synchronous-write workloads (databases,
VM disks with sync semantics) are where
ZFS earns its "slow" stories; the honest
fixes are workload placement (mirror
vdevs, a proper log device for the cases
that truly need it) and property
matching (chapter 5), not forum magic.
General file service on spinning disks
with a warm ARC is not one of ZFS's
problems, and claims either way without
your workload measured are marketing.
