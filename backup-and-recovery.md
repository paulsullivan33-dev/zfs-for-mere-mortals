# Backup and Recovery

Say it once, in bold, because every
other chapter quietly depends on it:
**a ZFS pool is not a backup, RAIDZ is
not a backup, and snapshots are not a
backup.** They all live on the same
disks, in the same box, under the same
roof as the original data. They protect
against disks failing and humans
fumbling. Backup is the thing that
protects against the box — deletion,
ransomware, theft, fire, flood, and the
lightning strike that arrives as a
power surge with opinions.

## The shape of a real backup

Chapter 7's send/receive is the
mechanism. A backup, built from it:

- **Another pool**, on other disks —
  ideally in another machine, better
  still in another building for the copy
  that matters most.
- **Fed by incremental sends** on a
  schedule, so being up to date costs
  minutes of traffic, not a weekend.
- **Verified** the way chapter 8 taught:
  the backup pool gets scrubbed too, so
  "we have a backup" means "we have a
  backup whose blocks have been read and
  checked lately".
- **Kept honest by a restore rehearsal**,
  at least once: receive a dataset onto
  spare space, open the files, prove the
  round trip works while nothing is
  actually on fire.

The classic pattern for a household:
the server's `tank` replicates nightly
to a `backup` pool on a second machine
(or a removable disk set that rotates
off-site), and the snapshots on the
source go back further than the
replication schedule, so "yesterday's
version" and "last month's version" are
both reachable.

## Automating snapshots and sends

By hand, chapters 6 and 7 are complete.
In practice nobody snapshots hourly by
hand for long, and the honest move is
to hand the routine to a tool built for
it. The well-known names in this space:

| Tool | Job |
|---|---|
| **sanoid** | Takes snapshots on a schedule with a retention policy (hourly/daily/monthly, keep N of each) |
| **syncoid** | sanoid's companion: runs the send/receive replication between pools or hosts |
| **znapzend** | One tool doing both planning jobs, configured per dataset |
| **zfs-auto-snapshot** | The older, simpler snapshot-only script many systems ship |

Whichever you choose, the discipline is
the guide's, not the tool's: snapshot
*before* the replication, keep the
anchor snapshot or bookmark the
incremental chain stands on, and make
failure loud (chapter 9's ZED habit
applies here too — a replication that
has failed silently for six weeks is a
story with a bad ending).

## Encryption, in one honest section

ZFS can encrypt datasets natively:
create with `-o encryption=aes-256-gcm
-o keyformat=passphrase`, unlock after
boot with `zfs load-key`, and the data
at rest is unreadable without the key.
Three facts to plan around. The key is
needed after every reboot before the
dataset mounts — somebody or something
must provide it. `zfs send -w` replicates
encrypted datasets *without decrypting
them*, so an off-site backup machine
never holds your key or your plaintext —
the strongest pattern this guide can
offer a household. And a lost key is
lost data, absolutely: keep the
passphrase in the same place you keep
the other thing your household cannot
afford to lose, which is not a sticky
note on the server.

## The recoveries, rehearsed

**"The server died."** Disks survive.
Put them in another ZFS machine (or
reinstall the OS around them) and
`zpool import` finds the pool exactly as
it was — this is why chapter 4 named
disks by ID. The pool is the portable
thing; the machine was always
replaceable.

**"I deleted the wrong dataset."**
`zfs destroy` is final — unless a
snapshot predates it *and was replicated
elsewhere*, or the dataset was being
received as a replica. Snapshot early,
replicate nightly; that pair of habits
is the whole answer.

**"The pool is gone entirely."**
Receive from the backup pool, in reverse,
onto new disks. Time is bounded by the
size of the data and the speed of the
disks, which is why the rehearsal
matters: you find out your restore takes
eleven hours on a calm Sunday, not
during the emergency.

**"Ransomware encrypted everything."**
The live datasets are ruined; the
snapshots are not (attackers changing
files do not rewrite snapshot history),
and the replica on the other machine is
untouched if the replication account
cannot delete there. Roll back or
receive from before the incident. This
scenario is the reason snapshots plus an
off-machine copy is a *security*
control, not just a convenience.

## The one-page discipline

1. Redundancy in the pool for disk
   failures (chapter 4).
2. Snapshots, automated, with retention,
   for human failures (chapter 6).
3. Replication, automated and verified,
   to another pool for machine failures
   (this chapter).
4. One restore, rehearsed, before you
   need it.
5. Keys and passphrases written down
   somewhere that survives the house's
   worst day.

Do those five and your storage story
stops depending on luck.
