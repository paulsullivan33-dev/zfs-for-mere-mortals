# Your First Pool

Two pools in this chapter: a **practice**
pool built from files, where every
command in this guide can be tried with
nothing at stake, and then the shape of a
**real** pool, with the decisions you
must make before you build it for real.

Do the practice pool first, even if you
aim to build the real thing today. ZFS
commands are unforgiving in one specific
way: they mostly do exactly what you
said, including when what you said was a
mistake.

## The practice pool: storage made of files

ZFS does not care whether a "disk" is a
disk. A plain file works, which makes
files the cheapest possible rehearsal
space. Make three 1 GB files:

```bash
mkdir -p ~/zfs-practice
cd ~/zfs-practice
truncate -s 1G disk1 disk2 disk3
```

(`truncate` makes *sparse* files — they
only take real space as the pool fills
them, which is why three "1 GB disks"
cost almost nothing on disk. Check with
`ls -lh` for the size they claim and
`du -h` for the space they use.)

Now build a pool from two of them, as a
mirror, leaving the third spare for
chapter 8's disk surgery:

```bash
sudo zpool create -f practice \
  $HOME/zfs-practice/disk1 \
  $HOME/zfs-practice/disk2
```

The `-f` is there because ZFS quite
reasonably hesitates to build a pool from
files in a home directory. On real disks
you will not need it, and should not use
flags that silence hesitation until you
know what the hesitation means.

Check what success looks like:

```bash
$ zpool status practice
  pool: practice
 state: ONLINE
config:
        NAME                       STATE
        practice                   ONLINE
          /home/you/zfs-practice/disk1  ONLINE
          /home/you/zfs-practice/disk2  ONLINE
errors: No known data errors
```

```bash
$ zfs list
NAME       USED  AVAIL  REFER  MOUNTPOINT
practice   104K  960M      24K  /practice
```

Your pool exists, and it is already a
mounted filesystem at `/practice`. Copy
some files into it. You are now running
real ZFS — checksums, copy-on-write, all
of it — on two imaginary disks. Chapters
5, 6 and 7 all work on this pool.

When you have finished practising, the
whole thing goes away cleanly:

```bash
sudo zpool destroy practice
rm ~/zfs-practice/disk*
```

## Building a real pool: the decisions

### Decision 1: which shape of vdev

Chapter 2's table is the whole menu. The
honest guidance for a home server:

- **Two disks → a mirror.** Simple, fast
  to repair (a replacement disk copies
  one surviving disk), and the easiest
  shape to grow later — you can swap
  both disks for bigger ones, one at a
  time, and the pool grows when the
  second swap finishes.
- **Four or more disks → RAIDZ2** is the
  usual sweet spot: any two disks can die
  and the pool survives, at a cost of
  two disks' worth of space.
- **RAIDZ1** (survive one disk) is widely
  considered thin ice for large modern
  disks, because rebuilding a failed
  disk means reading *every* remaining
  disk end to end — a second failure, or
  one unreadable spot, during that long
  rebuild is the classic way RAID5-style
  arrays die. The guide repeats the
  community's advice: mirrors for two,
  RAIDZ2 for a shelf of disks.
- **One disk** is allowed, and this guide
  won't pretend otherwise: checksums
  still catch corruption, but there is no
  second copy to heal from. Treat a
  single-disk pool as a filesystem with a
  great alarm system, and keep chapter
  10's backups close.

Whatever shape you choose, every vdev in
the pool should offer the same
protection. Adding one bare disk as a
new vdev to a mirror pool does not add
"a bit more space" — it adds a part of
the pool that kills everything when it
dies.

### Decision 2: whole disks, by ID

Give ZFS whole disks, not partitions,
and name them by their permanent IDs,
never by `sda`-style letters, which are
assigned in discovery order and can
shuffle between boots:

```bash
ls -l /dev/disk/by-id/
```

You are looking for entries like
`ata-WDC_WD40EFRX-XXXX_WD-WCC4...` — the
disk's own serial identity. A pool built
from by-id names can be *exported* and
*imported* on another machine and ZFS
will find its disks again whatever
letters they were given (chapter 8 uses
this; chapter 10 leans on it).

### Decision 3: the settings at creation

A few properties are worth setting at
birth, because changing them later only
affects new writes — or, for `ashift`,
cannot be changed at all:

```bash
sudo zpool create \
  -o ashift=12 \
  -O compression=lz4 \
  -O atime=off \
  -O xattr=sa \
  -O mountpoint=/tank \
  tank mirror \
  /dev/disk/by-id/<disk-one> \
  /dev/disk/by-id/<disk-two>
```

Plain meanings: `ashift=12` matches the
4K sectors of every modern disk (a wrong
guess here costs performance forever).
`compression=lz4` turns on compression
that is fast enough to leave on for
everything (chapter 5). `atime=off` stops
ZFS recording a timestamp every time a
file is merely *read*, which is write
traffic nobody reads. `xattr=sa` stores
extended attributes efficiently, which
Linux desktops and Samba appreciate.
`-O` sets those on the pool's root
dataset; `-o` sets pool-level options.

### Say it back, before you run it

`zpool create` makes no "are you sure?"
worth the name, and it will happily
destroy the contents of any disk you
name. Read your command aloud. Check the
IDs against the disks you meant. Then
run it, and check:

```bash
zpool status tank
zfs list
```

## Export, import: the pool is portable

A pool is not welded to the machine that
made it. `zpool export tank` writes
everything down and lets go; move the
disks to another ZFS machine and
`zpool import tank` picks the whole pool
up, datasets, snapshots and all. Keep
this in the back of your mind — it is
what turns "the server died" from a
disaster into an errand (chapter 10).
