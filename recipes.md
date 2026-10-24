# Recipes

Five ways real households put ZFS to
work. Each assumes chapters 3–5 are
done: an install, a pool, and datasets
planned on paper first. The settings are
starting points — chapter 5 explains
every property named.

## The photo archive that heals itself

**Shape:** two disks as a mirror;
dataset `tank/photos` with
`compression=lz4`, snapshots automated
hourly-then-daily, replicated nightly
(chapter 10) to a second machine.

**Why it works:** photos are the classic
irreplaceable data, and they barely
change once taken — so snapshots cost
almost nothing, the mirror repairs the
bit rot that quietly eats archives on
ordinary filesystems, and the scrub
(chapter 8) tells you once a month, in
writing, that every picture still
verifies. The dataset's `.zfs/snapshot`
folder also ends the "I deleted the good
one" panic at the level of single files.

## The media shelf that can grow

**Shape:** four disks as RAIDZ2;
`tank/media` with `recordsize=1M`
(big sequential files), `atime=off`,
and a quota that stops it eating the
space the rest of the pool needs.

**Why it works:** media is large,
replaceable, and streamed rather than
edited — the workload RAIDZ was born
for. Growth is a plan, not a crisis:
add a second RAIDZ2 vdev when the shelf
fills, or swap disks for bigger ones
two at a time. Snapshots here are
short-lived (a week) — media churn
makes old snapshots expensive, and
chapter 6's arithmetic is the leash.

## The backups landing zone

**Shape:** a dataset per machine backed
up — `tank/backups/laptop`,
`tank/backups/desktop` — receiving
send/receive streams or ordinary backup
tools' files, with a `quota` per machine
so one runaway backup cannot starve the
others, and `readonly=off` only during
its window if you're feeling strict.

**Why it works:** this is the dataset
design chapter 5 was written for: the
quotas are promises, the per-machine
split makes "restore just the laptop" a
one-dataset job, and the landing pool
itself is replicated or rotated off-site
per chapter 10. Label the physical
disks; a backups pool you cannot
identify in a dark cupboard is a
liability wearing a costume.

## The virtual machine shelf

**Shape:** datasets (or zvols) per VM on
mirrored SSDs if you have them;
snapshots taken *by the hypervisor's
schedule or by hand before every guest
upgrade*; the clone feature (chapter 6)
for trying changes on a copy.

**Why it works:** "snapshot the VM,
upgrade the VM, roll back the VM" is a
fifteen-minute loop instead of a
rebuild weekend, and clones make test
machines nearly free. One caution the
recipe must state: a snapshot of a
*running* VM is a crash-consistent
image — like pulling its plug — not a
clean shutdown. Quiesce the guest or
accept the plug-pull semantics
knowingly.

## The shared family drive, ZFS underneath

**Shape:** `tank/share` offered to the
household through Samba or NFS — either
by the file server normally, or with
ZFS's own `sharesmb` / `sharenfs`
properties registering the share for
you — with one child dataset per person
under it, each quoted, each snapshotted.

**Why it works:** the family sees an
ordinary shared drive; you see per-person
quotas, per-person snapshots ("Dad, I
deleted my coursework" → `.zfs/snapshot`,
thirty seconds), and checksums under
everything. This is ZFS at its most
invisible, which is where infrastructure
belongs.

## A habit that makes all five work

**Write the pool's map down.** One page:
which physical disk (serial number,
bay) is which by-id name, when each
entered service, what the datasets are
for. Keep it with the household handbook
(any wiki will do). Chapter 8's calm
afternoon depends entirely on this page
existing before it is needed.
