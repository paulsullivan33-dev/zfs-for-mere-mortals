# Installing ZFS

ZFS arrives differently on each platform,
and chapter 12's licence note is the
reason: Linux carries it as an add-on
kernel module, maintained by the OpenZFS
project and packaged by your
distribution, while FreeBSD includes it
in the base system. This chapter covers
the Linux route in full — it is the one
this guide's lab performed — and names
the others honestly at the end.

## What you are installing

Three layers, all from the one package
family:

- **The kernel module** (`zfs.ko`) — the
  filesystem itself, living in the kernel.
- **The tools** — `zpool` (manages pools)
  and `zfs` (manages datasets), plus
  helpers like `zed` (chapter 9).
- **The support services** — units that
  import pools at boot, mount datasets,
  and share them, in the right order.

Check your starting point:

```bash
uname -r                 # your kernel
cat /etc/os-release      # your distribution
zpool version            # already here? then skip to the checks
```

## Ubuntu and Debian

```bash
sudo apt update
sudo apt install zfsutils-linux
```

On **Ubuntu**, that is usually the whole
story: Ubuntu has shipped and supported
ZFS for years, and prebuilt modules come
with its kernels. On **Debian**, the
package comes from the `contrib`
component (make sure it is enabled) and
the module is *built on your machine* by
DKMS, which means two extra facts: the
matching kernel headers must be present
(`linux-headers-$(uname -r)`), and the
first install takes a few minutes while
it compiles. Read the output — a DKMS
build that fails is this chapter's
classic trap, and it fails in the log,
not at the prompt.

**Secure Boot is the second classic
trap.** A DKMS-built module is unsigned,
and a machine with Secure Boot on will
refuse to load it. Signs: the tools
installed fine, but everything answers
`The ZFS modules are not loaded`. The
fixes are the platform's usual ones —
enrol the DKMS signing key (MOK) as
Debian/Ubuntu's DKMS setup offers to do,
or use a distribution kernel that ships
the module prebuilt. Chapter 13 walks
the diagnosis.

## Loading and proving the module

```bash
sudo modprobe zfs
lsmod | grep zfs
zpool version
```

Check: `zpool version` prints the
OpenZFS version for both the userspace
tools and the kernel module — two lines
worth reading, because a mismatch
between them explains many small
weirdnesses and is worth recording in
your notes. Then the smallest possible
end-to-end proof, chapter 4's rehearsal
in one line:

```bash
sudo zpool create -f smoke \
  $(mktemp -d)/a $(mktemp -d)/b 2>/dev/null \
  || echo "see the file-backed pool in chapter 4"
```

Actually — don't be clever. Chapter 4's
practice pool *is* the end-to-end proof,
done properly with files you can see.
Install, run the checks above, and go
there.

## Boot-time behaviour, and other platforms

On Linux the installed services import
pools and mount datasets at boot; the
one thing to verify once is that your
pool is still there after a reboot —
`zpool list` when the machine comes
back. If it is missing, chapter 13 has
the cachefile story.

- **FreeBSD**: nothing to install. ZFS
  is in the base system, current, and
  the same commands in this guide apply,
  mutatis mutandis — it is the platform
  ZFS grew up on.
- **TrueNAS and friends**: ZFS with a
  web interface (chapter 12). The
  commands here remain true underneath
  the GUI, and are how you check its
  work.
- **macOS and Windows**: OpenZFS ports
  exist and are used, but they are young
  relatives, not the main line. This
  guide does not teach them.
- **In a container or restricted
  sandbox**: ZFS needs its kernel
  module, full stop. Where the module
  cannot be loaded, the tools install
  and then can do nothing — this guide's
  own lab meets exactly that wall, and
  its verification notes in `criteria.md`
  record precisely what could and could
  not be proven there. If you are
  evaluating ZFS, evaluate it on a real
  machine or a full virtual machine,
  never in a sandbox that can't load
  kernel code.

## Check: installed means loaded

```bash
zpool version            # both halves named
lsmod | grep -c zfs      # the module, counted
systemctl status zfs.target --no-pager
                         # boot services standing by
```

Three green lights, then chapter 4.
