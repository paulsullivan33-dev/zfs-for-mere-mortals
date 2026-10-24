# Snapshots, Clones and Rollback

A **snapshot** is a photograph of a
dataset at a moment: every file exactly
as it was, kept while the live dataset
moves on. Because of copy-on-write
(chapter 2), taking one costs no copying
at all — ZFS simply keeps the old
blocks instead of freeing them when new
writes arrive. This is the feature
people fall in love with, and the one
that quietly changes how brave you are
allowed to be: almost any mistake
followed by "I wish I could go back to
this morning" becomes reversible.

## Taking and using snapshots

```bash
sudo zfs snapshot tank/photos@before-import
zfs list -t snapshot
```

The name after the `@` is yours to
choose; tools that automate snapshots
(chapter 10) use dated names like
`tank/photos@2026-10-24-0600`.

**Looking inside without rolling back:**
a snapshot of a mounted dataset is
visible, read-only, in a hidden folder
inside it — `.zfs/snapshot/` at the
dataset's mountpoint. Yesterday's version
of one file is a plain copy away:

```bash
cp /tank/photos/.zfs/snapshot/before-import/cat.jpg \
   /tank/photos/cat-recovered.jpg
```

For a single file or a deleted folder,
this is the recovery you will actually
use, and it is worth more than any other
command in the chapter: **no restore
process, just files.**

**Rolling back** rewinds the whole
dataset to the snapshot, discarding
everything written since:

```bash
sudo zfs rollback tank/photos@before-import
```

Read that sentence again before running
it: *everything written since* —
including the good edits you made after
the bad one. Rollback also refuses to
cross a newer snapshot silently; if
snapshots exist after the target, ZFS
makes you destroy them first (`-r`),
which is the tool insisting you mean it.

**Destroying** a snapshot frees the
blocks that only it was keeping:

```bash
sudo zfs destroy tank/photos@before-import
sudo zfs destroy tank/photos@monday%friday
```

The second form destroys a *range* of
snapshots. Snapshot deletion is the
irreversible act in this chapter — a
snapshot cannot be snapshotted.

## What snapshots cost

A fresh snapshot uses essentially no
space. Its cost grows with *churn*: it
keeps every old block the live dataset
has since replaced or deleted. Numbers
to plan with:

- A quiet dataset (photos that never
  change) can keep a year of snapshots
  for almost nothing.
- A churning dataset (a VM image, a
  downloads folder) can make a week of
  snapshots cost as much as the dataset
  itself.

`zfs list -t snapshot` shows what each
snapshot is holding (`USED`), and `zfs
get usedbysnapshots tank/photos` totals
it. When a pool is mysteriously full,
old snapshots of a busy dataset are the
first place to look — chapter 13 agrees.

Snapshots are taken per dataset, and a
recursive flag keeps a family together:

```bash
sudo zfs snapshot -r tank@tonights-safety-net
```

That one command snapshots `tank` and
every dataset under it, all stamped with
the same transaction — the set you want
before surgery on the pool itself.

## Clones: a snapshot you can write to

A **clone** is a writable dataset born
from a snapshot:

```bash
sudo zfs clone tank/vm@clean-install tank/vm-test
```

The clone starts identical to the
snapshot and costs only its differences
as you change it — which makes it the
right tool for "let me try this upgrade
on a copy of the real VM". A clone stays
tied to its origin snapshot (you cannot
destroy the snapshot while the clone
lives) unless you **promote** the clone,
which swaps the relationship. Clones
that were meant to be temporary have a
way of becoming load-bearing; promote or
replace the ones you keep, so the
lineage stays understandable.

## Holds: protecting a snapshot from yourself

```bash
sudo zfs hold keep tank/photos@before-import
sudo zfs release keep tank/photos@before-import
```

A held snapshot refuses destruction —
including by the cleanup script at 3 a.m.
Put a hold on the snapshot you took
before the risky thing. It is the
difference between a safety net and a
note saying there used to be one.

## The habits that make this work

- **Snapshot before, not after.** Before
  upgrades, bulk renames, imports,
  imports especially. The command costs
  a second; the absence costs a weekend.
- **Name for the reason**, not the date,
  when a human takes it by hand:
  `@before-import` will explain itself in
  six months; `@snap23` will not.
- **Automate the routine ones** with a
  tool (chapter 10) and a retention rule —
  hourly for a day, daily for a week,
  weekly for a month is the common shape.
  Hoarded snapshots are a disk-full
  incident waiting for its moment.
- **Say the limit out loud:** snapshots
  live on the same pool as the data. They
  protect against mistakes and bad
  software. They do not protect against
  the pool dying, and chapter 10 exists
  because of that sentence.
