# Sending and Receiving

`zfs send` writes a dataset — or one of
its snapshots — out as a stream of bytes.
`zfs receive` reads such a stream and
rebuilds the dataset from it. Together
they are ZFS's built-in replication: the
way a pool's contents travel to another
pool, another machine, or a file in a
drawer. Almost everything in chapter 10
is these two commands wearing a coat.

## The basic move

```bash
sudo zfs snapshot tank/photos@send1
sudo zfs send tank/photos@send1 \
  | sudo zfs receive backup/photos
```

Run on one machine, that copies the
dataset (with its properties) into a
pool called `backup`. Split across
machines, the middle becomes a pipe
through SSH:

```bash
sudo zfs send tank/photos@send1 \
  | ssh you@other-server sudo zfs receive backup/photos
```

What arrives is not a pile of copied
files but the dataset itself — same
contents, same properties, same
snapshots. The receiving dataset is
read-only while it is a replica target,
which is the correct personality for a
backup: it cannot be casually edited
into uselessness.

## Incremental sends: only the difference

The first send must carry everything.
After that, ZFS can send *the difference
between two snapshots*:

```bash
sudo zfs snapshot tank/photos@send2
sudo zfs send -i tank/photos@send1 tank/photos@send2 \
  | sudo zfs receive backup/photos
```

This is the trick that makes replication
cheap enough to run nightly — or hourly —
over a home network: a day's changes to
a photo library are a trickle compared
to the library. The rule is strict: the
receiver must still hold the *from*
snapshot, untouched. Break that (someone
rolled the replica back, or a cleanup
ate the shared snapshot) and the next
incremental has nothing to stand on;
the fix is a fresh full send, which is
why replication scripts guard their
bookmarks and snapshots carefully.

**Bookmarks** are the lightweight
version of that anchor: a bookmark marks
a snapshot's position without keeping
its data pinned, so the source can
delete old snapshots and still send
incrementals from the bookmark.
`zfs bookmark tank/photos@send2
tank/photos#send2` — note the `#`. Tools
like syncoid (chapter 10) live on these.

## Variations worth knowing exist

| Flag | What it does | When you want it |
|---|---|---|
| `-R` | Send a dataset *and its children*, with their snapshots | Replicating a whole tree in one stream |
| `-p` | Include properties with the stream | Default behaviour varies by version; state it when it matters |
| `-x` | *Exclude* properties — receive with the receiver's own | Sending to a machine where paths or mountpoints differ |
| `-w` | Send raw: encrypted datasets stay encrypted in the stream | Backups of encrypted data that the backup machine must never be able to read |
| `-c` | Send compressed blocks as stored, no re-compressing | Saves CPU when both ends are ZFS |
| `-n -v` | Dry run: print what *would* be sent, and its size | Before the first big send, always |

Saving a stream to a file is allowed and
occasionally right — `zfs send tank/x@s
> x-stream.z` — but treat stream files as
what they are: a single opaque file that
is the whole dataset. A stream is not a
pile of files you can pick through; to
use any of it, you receive all of it.
Pools and pipes are the natural home of
send/receive; files are the emergency
exit.

## Receiving without tears

- The target dataset must not be in use,
  and receive will refuse to overwrite a
  dataset that has newer changes unless
  forced (`-F`, roll the target back to
  match). For a replica, forcing is the
  *point*; for anything else it is a
  loaded foot-gun. Know which you hold.
- A received dataset inherits the
  stream's mountpoint. On a machine
  where that path is already the live
  data, set the mountpoint aside first
  (`zfs receive -u` to not mount at all,
  then set properties deliberately).
- An interrupted send is resumable:
  ZFS can issue a **resume token**
  (`zfs get receive_resume_token`) so a
  nine-hour first replication that died
  at hour eight continues instead of
  restarting. Scripts use this; by hand,
  knowing it exists is enough.

## Check: did the copy arrive whole?

```bash
zfs list -t snapshot tank/photos backup/photos
diff <(cd /tank/photos && find . -type f | sort) \
     <(cd /backup/photos && find . -type f | sort)
```

Both snapshot lists should end at the
same snapshot name, and the file lists
should agree. The deeper check — that
every block on the receiving pool
verifies — is what a scrub does, and
chapter 8's scrub of the *backup* pool
is how replicas earn trust.
