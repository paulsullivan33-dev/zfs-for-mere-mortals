# Scrubs, Repair and Replacing a Disk

This is the chapter the whole design
points at: the routine that finds damage
before you do, and the afternoon a disk
dies and it turns out to be boring.

## The scrub: a full audit, in the background

A **scrub** reads every block in the
pool and checks it against its checksum,
repairing anything it can from the
redundant copies. It runs at low
priority while the pool serves normally
— you can scrub a pool in use, and
should:

```bash
sudo zpool scrub tank
zpool status tank        # watch progress, and the error counts
```

A scrub of a mostly-idle home pool takes
from minutes to some hours, depending on
how much *data* (not capacity) the pool
holds. The habit to build: **scrub on a
schedule** — monthly is the common
choice — because the scrub is what turns
"silent corruption" from this guide's
theory into a line in a status report
while there is still a good copy to
repair from. A pool that is never
scrubbed is a pool whose damage is
discovered at read time, which is to say
at the worst time.

The result you want, every time:

```
scan: scrub repaired 0B in 02:14:33 with 0 errors
errors: No known data errors
```

If it repaired something, that is the
system working — and a reason to look
hard at the disk that served the bad
block, starting with its SMART health
(the `smartmontools` guide is the
neighbour for that job).

## Reading `zpool status` like a professional

```
  pool: tank
 state: ONLINE
  scan: scrub repaired 0B ...
config:
        NAME        STATE     READ WRITE CKSUM
        tank        ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            <disk-1>  ONLINE     0     0     0
            <disk-2>  ONLINE     0     0     0
errors: No known data errors
```

Three error counters per device — READ,
WRITE, CKSUM. Occasional counts that stop
growing are often a cable or a controller
port rather than a dying disk; counts
that grow on every scrub are a disk
asking to be replaced. `DEGRADED` state
means the pool is running without full
protection: every day in DEGRADED is a
day the next failure is the real one.
`zpool status -x` prints one line —
"all pools are healthy" — or the details,
which makes it the right command for
scripts and chapter 9's checks.
`zpool clear tank` zeroes the counters
after you have fixed a cause, so new
errors stand out from old history.

## When a disk dies: replace, resilver

The pool is DEGRADED, one disk shows
FAULTED or has vanished. The procedure,
calmly:

1. **Identify the disk physically.**
   Match the by-id name in `zpool
   status` to a serial number on a
   drive label or in your notes (chapter
   11's recipes make this a habit
   *before* it matters). Pulling the
   wrong disk from a degraded mirror
   turns a repair into an obituary.
2. **Fit the replacement**, same size or
   larger.
3. **Tell ZFS:**

```bash
sudo zpool replace tank <old-disk-id> <new-disk-id>
zpool status tank
```

4. **Wait for the resilver.** The
   *resilver* is the rebuild: ZFS copies
   the live data (not the whole disk —
   only blocks in use) onto the new
   disk. `zpool status` shows progress
   and an estimate. The pool serves
   throughout, at somewhat reduced
   speed. When the resilver completes,
   the pool returns to ONLINE with full
   protection, and the incident is over.

If the old disk is completely dead and
gone, `zpool replace` with just the new
disk (after `zpool labelclear` caution
for reused disks) handles it — the
manual for your platform spells out the
variants; the shape is always *identify,
swap, replace, resilver, verify*.

A **spare** disk already fitted can be
told to step in (`zpool replace tank
<bad> <spare>`), or the pool can be
given `autoreplace=on` so a fresh disk
in the same slot is adopted
automatically. Automatic is lovely until
it adopts the disk you plugged in for
another purpose; most homes should
replace by hand and mean it.

## The practice pool earns its keep

Everything above can be rehearsed on
chapter 4's file-backed pool, and should
be, before a real disk ever fails:

```bash
sudo zpool offline practice $HOME/zfs-practice/disk2
```

Watch `zpool status` go DEGRADED, write
some files, then bring the third file
into play with `zpool replace`,
watch the resilver of a pool small
enough to finish in seconds, and finish
with `zpool status` ONLINE and clean.
The day the real pool degrades, your
hands will already know the moves —
which is the entire point of practising.

## What ZFS cannot repair

Honesty to end on. ZFS repairs from
*redundancy*: a second copy, parity, or
a dataset with `copies=2`. On a
single-disk pool a checksum failure is a
report, not a repair — `zpool status`
names the damaged file, and your only
recourse is chapter 10's backup. And no
amount of redundancy protects against
the pool-level accidents: wrong disk
pulled, wrong `destroy` typed, fire.
Redundancy is for disks failing, which
they do politely, one at a time, if you
scrub often enough to notice.
