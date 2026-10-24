# Troubleshooting

The order is the order you'll likely
meet them: install first, then pools,
then the slow realisations.

## "The ZFS modules are not loaded"

**Symptoms:** the tools are installed,
but every `zpool` command answers with
that sentence.

**Cause:** the kernel module isn't in
the running kernel. Three usual reasons,
in order: it was never built (DKMS
needed kernel headers that aren't
installed — read the install log for the
DKMS failure), it was built but Secure
Boot refuses the unsigned module, or the
environment cannot load kernel modules
at all (containers and restricted
sandboxes — check `ls /dev/zfs` and try
`sudo modprobe zfs` and read what it
says).

**Fix:** match the reason. Headers and a
DKMS rebuild for the first; enrol the
signing key (MOK) or use a kernel with
the module prebuilt for the second; for
the third there is no fix inside the
sandbox — ZFS is evaluated on a real
machine or a full VM. `zpool version`
after the fix should name both the tools
and the module.

## The pool is missing after a reboot

**Symptoms:** the machine comes back,
`zpool list` says *no pools available*,
but the disks are right there.

**Cause:** usually an import problem,
not data loss: the pool wasn't exported
cleanly on the way down, or the import
service ran before its disks appeared
(USB enclosures are famous for this).

**Fix:** look, then import:

```bash
sudo zpool import            # lists what can be found
sudo zpool import tank
```

If the pool was last used by a machine
that died without exporting,
`zpool import -f tank` says "I know,
import it anyway". If import claims the
pool is in use by another host — on a
shared-disk setup, believe it; on your
own desk, it means a previous life of
the same machine, and the warning is
still worth reading twice.

## "I deleted files and the space never came back"

**Symptoms:** you deleted 200 GB; `zfs
list` still shows the pool just as full.

**Cause:** snapshots are holding the
deleted data — which is snapshots
working as designed (chapter 6). Deleting
a file frees space only when no snapshot
still references it.

**Fix:** find the holder before
destroying anything:

```bash
zfs list -t snapshot -o name,used -s used
zfs get usedbysnapshots <dataset>
```

Then delete the snapshots you actually
mean to let go of — the retention policy
you chose in chapter 10 is the one to
argue with, not the emergency.

## The pool hangs at the worst moment: it's just full

**Symptoms:** writes stall or fail,
applications complain about space, `zpool
list` shows CAP in the high nineties.

**Cause:** chapter 2's warning come
true — copy-on-write with no room to
write into.

**Fix, in order of gentleness:** delete
or truncate something real (large files,
old datasets); expire snapshots you can
afford to lose (previous entry); move a
dataset off with chapter 7's send; or
add capacity — a new vdev of the right
shape, per chapter 4's sermon. Then
install chapter 9's routine so the next
time, you hear about it at 80%.

## A disk shows errors and you're not sure it's dying

**Symptoms:** CKSUM counts in
`zpool status`, climbing slowly; the
disk passes its own tests.

**Cause:** often not the disk at all —
cables, a hot HBA port, a loose
backplane connector, or marginal RAM
corrupting data before it is written.

**Fix:** reseat or swap the cable and
the port before buying anything; clear
the counters (`zpool clear tank`), run
a scrub, and see whether errors return
and *where* — errors that follow the
cable port indict the path, errors that
follow the disk indict the disk. Change
one thing at a time, or you'll never
know which it was. Chapter 8 covers
the replacement once the verdict is in.

## `zfs` commands take seconds each

**Symptoms:** even `zfs list` pauses.

**Cause:** almost always a pool member
misbehaving (a disk retrying reads
forever), or a snapshot list in the tens
of thousands on busy datasets.

**Fix:** `zpool status` and `zpool
iostat -v` to find the device taking
seconds to answer; deal with it per the
previous entry. For snapshot sprawl,
`zfs list -t snapshot | wc -l` tells the
truth, and chapter 6's retention is the
cure.

## `zpool create` says the device is in use / contains a filesystem

**Symptoms:** creation refuses a disk
you believe is empty.

**Cause:** old signatures survive —
a previous filesystem, RAID metadata,
or a partition table from the disk's
last life. ZFS is being polite.

**Fix:** be certain the disk is the one
you think it is (serial, size, bay),
then wipe its signatures
(`wipefs -a <disk>` is the honest tool)
and retry. `-f` silences the hesitation;
use it after you have done the checking
by eye, never instead of it. This entry
is why chapter 4 reads the IDs aloud.
