## Haramball — Forensics 

**Files:** `simeone.jpg`

> Simeone has something to hide.  
> Look beyond the picture itself. The truth might be hidden in plain sight.

### Solve

Check the image’s metadata:

bash

```bash
exiftool simeone.jpg
```

Output includes:

```
Comment                         : 0SSEC{W3lC0M3_B4LL3RS}
```

### Flag

```
0SSEC{W3lC0M3_B4LL3RS}
```

### Explanation

The flag is stored as a JPEG comment (COM segment), readable with any metadata tool — `exiftool`, `identify -verbose`, or `strings`. No encoding, no extra steps.

### Takeaways

- Always check EXIF/metadata early in forensics challenges — it’s a one-command check.
- `exiftool` is the standard tool; ImageMagick’s `identify -verbose` and even `strings` will also surface a plain COM segment.

---