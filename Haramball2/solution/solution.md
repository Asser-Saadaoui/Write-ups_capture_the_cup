## Haramball 2 — Forensics (150 pts)

**Files:** `Arteta.jpg`

> Mikel looks calm, but there’s more going on under the surface.  
> One glance won’t be enough — dig deeper.

### Solve

#### 1. First look

bash

```bash
exiftool Arteta.jpg
```

```
Comment                         : 0SSEC{DIG_D33P3R}
```

Submitting this doesn’t work — it’s a decoy, and the text itself (“dig deeper”) is the hint that there’s more.

#### 2. Dig deeper

JPEGs can contain multiple comment (COM) segments, but `exiftool` only shows the first one by default. The `-a` flag reveals all duplicate tags:

bash

```bash
exiftool -a Arteta.jpg
```

```
Comment                         : 0SSEC{DIG_D33P3R}
Comment                         : 0SSEC{PARK_TH3_BUS}
```

### Flag

```
0SSEC{PARK_TH3_BUS}
```

### Explanation

The file has two `FF FE` (COM) marker segments stacked back-to-back right after the JPEG header. Most tools show only one of them by default:

- `exiftool` (no flags) → shows the **first** COM segment
- `exiftool -a` → shows **all** of them, in order
- `identify`/`file` (ImageMagick/libjpeg) → show the **last** one instead

So depending on which tool a player reaches for, they may see either flag as “the” comment — the inconsistency itself nudges players to check more than one tool, or to specifically reach for `-a`.

### Takeaways

- Metadata tools don’t always show _everything_ — `-a` in exiftool is the key flag for duplicate/repeated tags.
- A decoy flag that still reads as a hint (“dig deeper”) is a nice way to keep players moving instead of getting stuck submitting a wrong answer blind.
- Different tools can report different values for the same field when a file isn’t well-formed by spec — useful to know generally, not just for CTFs.

---

### Author notes: building the challenge files

Both files are generated from clean source photos with a small script that inserts raw JPEG COM segments:

python

```python
import struct

def make_com_segment(data: bytes) -> bytes:
    length = len(data) + 2  # length field includes itself, not the FF FE
    return b"\xff\xfe" + struct.pack(">H", length) + data

def insert_comments(infile, outfile, comments: list[bytes]):
    d = open(infile, "rb").read()
    assert d[:2] == b"\xff\xd8"  # must start with SOI
    segs = b"".join(make_com_segment(c) for c in comments)
    new_data = d[:2] + segs + d[2:]  # insert right after SOI
    open(outfile, "wb").write(new_data)

# Haramball 1: one comment, flag in plaintext
insert_comments("source1.jpg", "simeone.jpg", [b"0SSEC{W3lC0M3_B4LL3RS}"])

# Haramball 2: two comments — decoy first, real flag second
insert_comments("source2.jpg", "Arteta.jpg", [
    b"0SSEC{DIG_D33P3R}",
    b"0SSEC{PARK_TH3_BUS}",
])
```