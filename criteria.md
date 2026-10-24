# Writing Criteria

Every guide in the "for Mere Mortals" series is written
against the same nine points. They exist so this guide
feels like the others and can be trusted the same way.

1. **Audience and prerequisites stated.** The reader
   knows, before starting, what they need, what they are
   assumed to already know, and whether this guide is for
   them at all.
2. **One canonical path.** Each task is taught one way —
   the way that works — not as a survey of every way it
   could be done. Alternatives are mentioned only where
   choosing wrongly would hurt.
3. **Verified, not transcribed.** Commands and claims are
   tested where testing is possible. Where it is not,
   the guide says so plainly and names what was checked
   instead — it never silently passes off documentation
   as experience.
4. **Mental model before mechanics.** The ideas come
   first; commands are then read as expressions of the
   model, so failures can be reasoned about rather than
   merely searched for.
5. **Failure is a curriculum topic.** Install failure
   modes, realistic things going wrong, and their
   diagnoses get their own chapter — because that is when
   a guide earns its shelf space.
6. **Permanent fixtures.** The examples are written for
   general audiences and durable use, never modelled on
   any one person's organisations, events, or life.
7. **Realistic scope.** A guide covers what a home or
   small-network operator actually needs, and says what
   it leaves out rather than pretending to be complete.
8. **Consistent apparatus.** Short chapters, check
   blocks, plain words for things before their proper
   names, recipes that recombine the taught parts, a
   glossary, and shipping lab notes.
9. **Honesty about the subject.** Limits get a chapter of
   their own. A tool's real trade-offs — including when
   a neighbouring tool is the better answer — are stated
   without salesmanship.

## Verification notes

The lab (Ubuntu 24.04.5, kernel 7.0.0-39-generic, sandbox,
2026-10-24) verified the OpenZFS **userspace only**. The
straight `apt-get install zfsutils-linux` was blocked by
an unrelated package transition elsewhere in the shared
sandbox (a fuse/fuse3 conflict; transcript
`t01a-install-attempt.txt`), so the distribution's own
`zfsutils-linux` 2.2.2-0ubuntu9.5 packages were downloaded
and their binaries run unpacked — the tools exercised are
the real ones, but no completed dpkg install is claimed.
The kernel module could not be loaded in the sandbox
(`modprobe zfs`: module not found for this kernel;
`/dev/zfs` absent), so the file-backed pool of chapter 4
was attempted and refused by the tools themselves
(transcript `t04-pool-attempt.txt`).

**Verified live** (transcripts in
`zfs-lab/transcripts/`): the current package versions
from the Ubuntu archive (`t00`); `zpool --version` /
`zfs --version` output, zfs-2.2.2-0ubuntu9.5 (`t01`); the
complete `zpool` and `zfs` --help command surfaces, which
every command in the guide was checked against (`t02`);
the exact module-missing behaviour the troubleshooting
chapter quotes (`t03`, `t04`).

**Documented-only:** everything requiring a loaded
module — pool creation, import and export; datasets and
property inheritance; snapshots, clones and rollback;
send/receive and bookmarks; scrubs and disk replacement;
ZED and live monitoring. Those chapters were written
from the OpenZFS 2.2 man pages and project documentation,
and their syntax cross-checked against the installed 2.2.2
help surface, but no behaviour in them was observed. The
README repeats this ceiling so no reader depends on the
distinction being buried here.
