# Datasets and Properties

A pool is one bucket. **Datasets** are
how you organise what's in the bucket —
one dataset per job, each with its own
settings, its own snapshots, and its own
space rules. This chapter is mostly
properties: the small knobs that turn a
dataset from "a folder" into "a folder
that fits its job exactly".

## Making datasets

```bash
sudo zfs create tank/photos
sudo zfs create tank/media
sudo zfs create tank/backups
sudo zfs create tank/photos/2026
```

Datasets nest like folders, mount
automatically under the pool's
mountpoint (`/tank/photos`, and so on),
and can be created and destroyed without
touching the pool or each other:

```bash
$ zfs list
NAME              USED  AVAIL  REFER  MOUNTPOINT
tank              212G   700G    96K  /tank
tank/backups      120G   700G   120G  /tank/backups
tank/media         80G   700G    80G  /tank/media
tank/photos        12G   700G    12G  /tank/photos
tank/photos/2026    2G   700G     2G  /tank/photos/2026
```

Notice `AVAIL` is the same everywhere:
without limits, every dataset draws on
the whole pool's free space. That is the
point of a pool — and the reason for the
quotas below, when one dataset's appetite
must not starve the others.

## Properties: set once, inherited below

Read any property with `zfs get`, set it
with `zfs set`, and undo a local setting
with `zfs inherit`:

```bash
zfs get compression tank/photos
sudo zfs set compression=lz4 tank
zfs get -r compression tank   # -r: show the whole tree
```

The output tells you the *source* of the
value — `local` (set here), `inherited
from tank` (set by a parent), or
`default`. This is the property system's
one rule: **a setting on a parent applies
to everything under it, unless a child
overrides it.** Set your opinions at the
pool root once, and make exceptions
dataset by dataset.

Two cautions that save real confusion.
Most property changes apply only to
*new* writes — setting compression on a
dataset full of old files does not
recompress them (copy the files afresh
if you want that). And a handful of
properties are fixed at dataset creation
(the big one, `recordsize`, can be
changed, but again only reshapes new
writes).

## The properties worth knowing

| Property | Plain meaning | Home guidance |
|---|---|---|
| `compression` | Compress blocks as they are written | `lz4` everywhere. It is fast, it spots incompressible data and gives up cheaply, and smaller data is also faster data |
| `atime` | Record when files are read | `off` (set at pool creation) — almost nobody needs read-times |
| `recordsize` | The block size ZFS thinks in for a dataset | Leave at the 128K default, except: 1M suits datasets of huge files (media, disk images), and small values suit databases. Match the workload, not the rumour |
| `quota` | A hard ceiling on a dataset's size | Use on the dataset that might eat the pool — a backups target, a child's media folder |
| `reservation` | Space guaranteed to a dataset | The opposite promise: this much is held for this dataset no matter what the others do |
| `copies` | Keep 2 (or 3) copies of this dataset's blocks | A way to buy extra safety for one precious dataset on an otherwise single-disk pool. Costs the space, obviously |
| `readonly` | Refuse writes | Handy for a finished archive nobody should "tidy" |
| `mountpoint` | Where the dataset mounts | `none` for parents you use purely as containers; `legacy` if another tool (fstab, a container runtime) should mount it |
| `canmount` | Whether this dataset itself mounts | `noauto` or `off` for container parents, so the parent doesn't shadow its children |
| `sync` | How strictly writes wait for the disk | Leave `standard`. `disabled` is faster and can lose the last seconds of writes in a crash — a fine trade for a scratch dataset, a terrible one by accident |
| `dedup` | Store identical blocks once | **Off.** It needs large amounts of RAM relative to pool size and saves most people almost nothing. The one property this guide flatly advises against |
| `sharenfs` / `sharesmb` | Offer the dataset over the network | A convenience that asks ZFS to register the share for you. Chapter 11 uses it in one recipe |

## A worked shape

A typical home server's tree, and why:

```
tank
├── media        compression=lz4, recordsize=1M
├── photos       compression=lz4, copies default
├── documents    the small, precious stuff
├── backups      quota set, receives other machines
├── vm           recordsize left small-ish, sync as the guest needs
└── scratch      sync=disabled, nobody cries if it vanishes
```

The pattern to copy is not the names —
it is that each dataset exists because
its job differs, and its properties say
how. When every dataset is identical,
you didn't need datasets; when each is
tuned to its contents, snapshots and
quotas (the next two chapters) become
per-job tools instead of blunt ones.

## Check: what did I just change?

```bash
zfs get all tank/photos | head -30
zfs get -r -o name,property,value,source all tank
```

The second command is the audit view:
every non-default value in the tree, and
where it came from. If you cannot
explain a line in it, something was set
by accident — `zfs inherit <property>
<dataset>` puts it back to following
its parent.
