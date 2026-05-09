# Linux Basics for Hackers
## Module 9 — Archiving & Compression

---

## What this module is about

Sometimes you need to move a bunch of files as one package. Sometimes you need to shrink a large file before sending it somewhere. And sometimes — in forensics or serious recon — you need an exact, bit-for-bit copy of an entire disk. This module covers all three: archiving with `tar`, compressing with `gzip` and `bzip2`, and low-level disk imaging with `dd`.

---

## Archiving vs compression — not the same thing

These two words get used interchangeably but they mean different things.

**Archiving** means combining multiple files into one single file. It doesn't make the files smaller — it just bundles them together. The tool for this in Linux is `tar`.

**Compression** means making a file smaller by encoding it more efficiently. The tools for this are `gzip` and `bzip2`. They work on single files.

In practice you usually do both at once — archive first, then compress the result. That's where `.tar.gz` files come from. The `.tar` part means it was archived, the `.gz` part means it was then compressed with gzip.

---

## tar — bundling files together

`tar` stands for "Tape Archive" — the name comes from when backups were stored on magnetic tape. The name is old, the tool is still everywhere.

**Creating an archive:**

```bash
tar -cvf archive.tar file1.txt file2.txt file3.txt
```

Breaking down the flags:
- `-c` — create a new archive
- `-v` — verbose, show what's being added (optional but useful)
- `-f` — the next argument is the filename of the archive

The result is `archive.tar` — one file containing all three.

**Extracting an archive:**

```bash
tar -xvf archive.tar
```

- `-x` — extract
- `-v` — verbose
- `-f` — file to extract from

This puts the files back in your current directory.

**Listing contents without extracting:**

```bash
tar -tvf archive.tar
```

- `-t` — list contents

Useful when someone hands you a tar file and you want to see what's inside before unpacking it.

---

## gzip and gunzip — making files smaller

`gzip` compresses a single file in place — the original disappears and gets replaced by a `.gz` version.

```bash
gzip archive.tar
```

Result: `archive.tar` is gone, replaced by `archive.tar.gz`.

To decompress:

```bash
gunzip archive.tar.gz
```

Or equivalently:

```bash
gzip -d archive.tar.gz
```

Both do the same thing — restore the original file.

---

## Doing both at once — tar.gz

Instead of running `tar` then `gzip` separately, you can do it in one command with the `-z` flag:

**Create a compressed archive:**

```bash
tar -cvzf archive.tar.gz file1.txt file2.txt file3.txt
```

The `z` added to the flags means "also compress with gzip."

**Extract a compressed archive:**

```bash
tar -xvzf archive.tar.gz
```

Same idea — `x` to extract, `z` to handle the gzip layer.

This is the format you'll see most often. When someone says "tar-ball" they usually mean a `.tar.gz` file.

---

## bzip2 — an alternative to gzip

`bzip2` is another compression tool. It generally produces smaller files than gzip but takes longer to compress and decompress. For most purposes the difference is small, but it's worth knowing both exist.

| | gzip | bzip2 |
|---|---|---|
| Speed | Faster | Slower |
| Compression ratio | Good | Better |
| File extension | `.gz` | `.bz2` |
| tar flag | `-z` | `-j` |

With tar:

```bash
tar -cvjf archive.tar.bz2 files/
```

Extract:

```bash
tar -xvjf archive.tar.bz2
```

The only difference is `-j` instead of `-z`.

---

## dd — low-level disk copying

`dd` is in a completely different category from the other tools here. While `tar` and `gzip` work with files and folders, `dd` works at the raw disk level. It copies data block by block, byte by byte, without caring about the filesystem structure at all.

```bash
dd if=/dev/sda of=/dev/sdb
```

- `if` — input file (the source)
- `of` — output file (the destination)

This copies every single bit from `/dev/sda` (your first hard drive) to `/dev/sdb` (a second drive). The copy is perfect — it includes deleted files, filesystem metadata, everything. The destination drive becomes an exact clone of the source.

**Making a disk image file:**

```bash
dd if=/dev/sda of=disk_image.img
```

This saves the entire disk as a file instead of copying it to another drive. Forensics investigators do this so they can work on the image without touching the original evidence.

**Why it's called "Data Destroyer":**

If you swap `if` and `of` by accident — or target the wrong drive — you overwrite a real disk with zeros or garbage. It doesn't ask for confirmation. There's no undo.

```bash
dd if=/dev/sdb of=/dev/sda    ← overwrites your main drive with whatever is on sdb
```

Always double-check your `if` and `of` values before running dd. Triple-check if one of them is a real disk.

**Adding a progress indicator:**

By default `dd` runs silently. You have no idea how far along it is. Add `status=progress` to see output:

```bash
dd if=/dev/sda of=disk_image.img status=progress
```

---

## The forensics angle

`dd` is a standard tool in digital forensics. When investigators seize a computer, they don't work directly on the original drive — they make a `dd` image of it first, then analyze the image. This way the evidence is never touched or altered.

Because `dd` copies at the block level, it captures everything: files, deleted files, file fragments, filesystem structure, slack space. Tools like Autopsy and Sleuth Kit can then analyze the image and recover data that would be invisible to a normal file browser.

---

## Command reference

| Command | What it does |
|---|---|
| `tar -cvf archive.tar files` | Create a tar archive |
| `tar -xvf archive.tar` | Extract a tar archive |
| `tar -tvf archive.tar` | List contents of a tar archive |
| `tar -cvzf archive.tar.gz files` | Create a gzip-compressed archive |
| `tar -xvzf archive.tar.gz` | Extract a gzip-compressed archive |
| `tar -cvjf archive.tar.bz2 files` | Create a bzip2-compressed archive |
| `tar -xvjf archive.tar.bz2` | Extract a bzip2-compressed archive |
| `gzip filename` | Compress a file with gzip |
| `gunzip filename.gz` | Decompress a .gz file |
| `dd if=source of=destination` | Copy raw blocks from source to destination |
| `dd if=/dev/sda of=image.img status=progress` | Create a disk image with progress output |

---

## tar flags at a glance

| Flag | Meaning |
|---|---|
| `-c` | Create a new archive |
| `-x` | Extract from an archive |
| `-t` | List contents |
| `-v` | Verbose output |
| `-f` | Specify the filename |
| `-z` | Use gzip compression |
| `-j` | Use bzip2 compression |

---

## Practice

- Create three empty text files with `touch file1.txt file2.txt file3.txt`
- Bundle them into a tar archive: `tar -cvf bundle.tar file1.txt file2.txt file3.txt`
- Compress it: `gzip bundle.tar` — then run `ls -lh` and see the size difference
- Extract it back out with `tar -xvzf bundle.tar.gz` and confirm the files are there
- Try the same thing with `bzip2` and compare the final file sizes between `.tar.gz` and `.tar.bz2`

---

*Up next: Module 10 — Filesystem & Storage Devices*
