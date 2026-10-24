# What ZFS Is

ZFS is two things that used to be separate
jobs, done by one piece of software: it is
a **filesystem** (the thing that organises
your files into folders) and a **volume
manager** (the thing that decides how
disks combine into storage). You hand ZFS
some disks, it joins them into a *pool*,
and you carve the pool into *datasets* —
folders with superpowers. Your files live
in the datasets.

It was built at Sun Microsystems in the
2000s for servers holding data people
could not afford to lose, and it is now
the standard storage on FreeBSD and
TrueNAS, and a first-class option on
Linux through the OpenZFS project. What
makes it different from the filesystems
you already know is not speed or features.
It is a single, stubborn idea:

**ZFS assumes your disks are lying to
you, and it checks.**

## The three promises

Everything in this guide hangs off three
design decisions. Ordinary filesystems
make none of them.

**1. Every block is checksummed, end to
end.** When ZFS writes a block of your
data, it stores a checksum of that block
*somewhere else* — in the metadata that
points to it, never beside it. When the
block is read back, the checksum is
recalculated and compared. A disk that
quietly returns wrong data — bit rot, a
flaky cable, a controller having a bad
day — is *caught*, not trusted. Silent
corruption is the enemy ZFS was designed
around; on a conventional filesystem the
same corruption is copied faithfully into
your backups before anyone notices.

**2. Nothing is ever overwritten in
place.** ZFS writes new data to fresh
space, then atomically updates the
pointer to it. This is called
*copy-on-write*. Pull the plug mid-write
and you get the old version or the new
one, never a half-written mixture — so a
ZFS pool never needs the long filesystem
check (fsck) that other filesystems run
after a crash. The same trick makes
snapshots almost free (chapter 6): a
snapshot is just the old set of pointers,
kept.

**3. Given a second copy, ZFS heals.**
If a block fails its checksum and the
pool holds another copy — because the
disks are mirrored or parity-protected —
ZFS returns the good copy to you *and
repairs the bad one in the background*.
You may never know it happened, except
that the pool's status keeps count
(chapter 8). Detection without redundancy
tells you a file is damaged; redundancy
is what turns that sad knowledge into a
non-event.

## What it is not

- **Not a backup.** A pool protects
  against disk failure and silent
  corruption. It does nothing against
  deletion, ransomware, fire, or theft —
  yesterday's snapshot is on the same
  disks as today's mistake. Chapter 10
  is blunt about this.
- **Not magic against bad planning.** A
  pool made of one disk can detect
  corruption but cannot repair it. The
  redundancy decision is made when the
  pool is created, and some of it cannot
  be changed later (chapter 4).
- **Not the fastest at everything.** ZFS
  spends CPU and memory on checksums and
  caching. On roomy hardware you will
  never notice. On a starved machine you
  will, and chapter 12 says so plainly.
- **Not a toy to point at disks you care
  about.** Creating a pool destroys what
  is on the disks it takes. This guide
  starts with a practice pool made of
  *files*, precisely so your first
  mistakes are free (chapter 4).

## Who it is for

The household or small-server keeper with
a box of disks and data that matters: the
photo archive, the media library, the
family documents, the backups of every
other machine. ZFS suits that reader
because its safety features are not a
mode you enable — they are how it works
all the time, including while you are on
holiday and not thinking about storage
at all.

## The one-sentence version

Give ZFS two or more disks and it will
watch every block it stores, tell you
when one goes bad, fix it from the other
copy, and hand you time travel
(snapshots) as a side effect — in
exchange for some memory, some planning
up front, and honesty about what it
cannot do.
