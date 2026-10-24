# Watching the Pool

ZFS is unusually willing to tell you how
it is. The trick is to look *routinely*,
not just when something smells wrong —
most pool tragedies were announced weeks
earlier in a status nobody read.

## The three commands of routine

```bash
zpool status -x        # one line if all is well
zpool list             # capacity and health at a glance
zfs list               # who is using the space
```

`zpool list` carries the two numbers to
watch over months: **CAP** (how full)
and **FRAG** (fragmentation). Fullness
is the one with teeth — recall from
chapter 2 that copy-on-write needs free
space to work in, so a pool heading past
roughly 80% full deserves a plan, not a
shrug. The plan is one of: bigger disks,
a new vdev, quotas on the greedy
(datasets, chapter 5), or evicting the
archives to chapter 10's backup pool
where they were always meant to live.

## Events: the pool keeps a diary

```bash
zpool events           # recent pool events
zpool events -v        # with all the detail
```

Every fault, scrub, resilver and repair
is written here with timestamps. When
someone says "the disk died Tuesday,"
the events log is how you find out it
actually started throwing checksum
errors in September.

On Linux, the **ZED** (ZFS Event
Daemon, from the `zfs-zed` package)
watches that diary for you and can run
scripts or send mail on the events that
matter — a device faulting, a scrub
finishing with errors, a pool degrading.
Configuring ZED to send one honest email
on those events is, hour for hour, the
highest-value monitoring work in this
guide: it converts "check the pool
sometimes" into "the pool speaks when
it needs you".

## Space accounting that answers questions

When the pool fills and "but I deleted
loads" — chapter 13's favourite mystery
— these are the spades:

```bash
zfs list -o space             # used by data, snapshots, children
zfs get usedbysnapshots tank/media
zfs list -t snapshot -o name,used -s used
```

`usedbysnapshots` is the usual culprit:
deleting a file that a snapshot still
holds frees nothing until the snapshot
goes too. Second culprit: a dataset
quota or a `refreservation` holding
space hostage on paper. Third:
compression ratios lying by omission —
`zfs get compressratio` shows what a
dataset's data *would* occupy.

## The ARC and performance, briefly

If the server feels slow, look before
you tune:

```bash
arc_summary | head -40        # cache size, hit rate (where installed)
zpool iostat -v 5             # per-vdev traffic, sampled every 5s
zpool iostat -q               # queue depths: is a disk the bottleneck?
```

An ARC hit rate in the high nineties
means memory is doing its job. A single
vdev pegged while others idle is
unbalanced storage, not a CPU problem.
Beyond this, tuning ZFS is a hobby with
forums of its own; the guide's stance is
that defaults on sane hardware are the
right answer for a home pool, and the
counters above are for proving a problem
exists before fixing it.

## SMART: the disks' own testimony

ZFS knows about data it was handed; the
disks know about themselves — reallocated
sectors, pending sectors, temperature,
hours. `smartctl -a <disk>` (from
smartmontools) read alongside `zpool
status` gives the full picture: ZFS
says *this block failed*, SMART says
*whether the disk is generally dying*.
A disk with growing checksum errors
*and* growing reallocated sectors is not
a mystery; it is a shopping trip.

## A minimal routine

Put it together as a monthly five
minutes, or automate exactly this:

1. `zpool status -x` — healthy?
2. Scrub ran this month, finished clean?
3. CAP under 80%, FRAG not climbing?
4. Snapshot schedule producing snapshots
   (chapter 10), and the newest replica
   is as fresh as you think it is?
5. One glance at SMART for the oldest
   disk in the pool.

Five minutes, monthly, is the whole
price of never being surprised by your
storage. The rest of this guide exists
so that when one of those five answers
is "no", you know what the next command
is.
