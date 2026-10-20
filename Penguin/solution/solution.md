
**Files:** `penguin.jpg`

### Solve

Run `strings` on the image and grep for the flag format:

bash

```bash
strings penguin.jpg | grep SSEC
```

Output:

```
D0SSEC{W3lc0m3_t0_CC}
```

The leading `D` is a stray printable byte from the image data. The flag starts at `0` (zero).

### Flag

```
0SSEC{W3lc0m3_t0_CC}
```

### Explanation

The flag is stored as plain ASCII inside the JPEG scan data, so `strings` finds it directly. `exiftool` and `binwalk` find nothing, because it isn’t in the metadata or appended after the end of the file.