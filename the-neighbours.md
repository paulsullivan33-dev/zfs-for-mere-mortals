# The Neighbours

ZFS is one answer to "how should my
disks be organised?" These are the
others, and what each trades. Nothing
here is a ranking — the right answer is
the one whose costs your setup can pay.

| Tool | Shape | Choose it when |
|---|---|---|
| **ext4** | The default Linux filesystem: one filesystem per disk or partition | Simplicity wins: one disk, no redundancy wanted, checksums of metadata only are enough. The baseline everything else is measured against |
| **XFS** | Linux filesystem built for very large files and parallel workloads | You need ext4's simplicity at bigger scale, and redundancy lives elsewhere (hardware RAID, the application) |
| **Btrfs** | Linux's other copy-on-write filesystem, in the kernel: snapshots, checksums, subvolumes, built-in RAID | You want ZFS's *ideas* with mainline-kernel convenience. Its RAID5/6 modes have a long-standing write-hole caveat the community will tell you about at length; its mirror and single modes are widely used |
| **LVM + mdadm + ext4/XFS** | The classic layered stack: software RAID below, logical volumes, a plain filesystem on top | You want each layer standard, separately understood, and repairable with tools every Linux admin already has — and you accept that the filesystem cannot verify data the RAID layer handed it |
| **SnapRAID + MergerFS** | Parity snapshots over plain disks, pooled into one tree (a companion guide in this series covers each) | Media that barely changes, disks of mixed sizes added one at a time, and a strong preference for files staying readable as ordinary files on ordinary filesystems |
| **Hardware RAID card** | A controller presents an array as one disk; a plain filesystem on top | Rarely, at home, any more: the array dies with the card unless you keep a twin, and the filesystem above it still cannot tell when the card hands it wrong data |
| **TrueNAS / a storage appliance OS** | ZFS underneath, managed through a web interface | You want exactly this guide's storage with the command line replaced by a thoughtful GUI, and a box whose only job is storage |

ZFS's own trade is visible in the table:
it asks for the whole stack — disks,
volumes and filesystem decided together,
up front — and for enough memory to
cache well. In exchange you get the
thing none of the layered answers can
offer: the filesystem itself knows
whether the data is right, because the
checksums live above the disks, not
below them. Btrfs is the honest
exception that shares the idea; its
crowd and ZFS's crowd have been
arguing productively for years, and a
reader served by either is well served.

Two closing notes. **Switching costs are
real**: there is no in-place conversion
to ZFS; your data arrives by copying,
and leaves the same way (chapter 7's
send is ZFS-to-ZFS only). And whatever
you choose, chapter 10's question
applies unchanged — where is the copy
that survives the machine, and when did
you last rehearse bringing it back?
