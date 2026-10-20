# Vintage: Writeup

**Category:** Forensics | **Tags:** stego, forensics, wine, reversing | **Author:** Trafalgar

## Challenge

A glass, a label, and a bottle that was never meant to be opened on Linux.

Not everything in this picture is a picture.

Given: `wine_challenge.jpg`, an image of the Wine logo.

## Solution

### 1. Triage the image

```bash
file wine_challenge.jpg
exiftool wine_challenge.jpg
binwalk wine_challenge.jpg
```

It's a normal 208x329 JPEG with nothing in the metadata. The file is about 10 KB, though, which is a lot for an image this small. The JPEG data itself ends after about 4 KB, so something else is sitting in the file.

### 2. Find the appended data

```bash
strings -n 6 wine_challenge.jpg | tail
```

The last line shows `ctf.exe` (followed by `UT`, a ZIP timestamp field). That is a filename from a ZIP central directory. A ZIP is read from the end of the file, so it can be appended to a JPEG without breaking either format.

### 3. Extract it

```bash
unzip wine_challenge.jpg
```

`unzip` warns about "extra bytes at beginning" (the JPEG data) and extracts `ctf.exe`.

### 4. Identify the payload

```bash
file ctf.exe
# PE32+ executable (console) x86-64, for MS Windows
```

A Windows binary, so this is the Wine hint.

### 5. Get the flag

Run it under Wine:

```bash
wine ctf.exe
```

**Don't trust `strings` here.** This works, but the string it finds is a decoy:

```bash
strings -n 5 ctf.exe | grep SSEC
# 0SSEC{str1ngs_1s_n0t_3n0ugh}
```

That flag is never printed by the program. The real flag is not stored in the file as text.

### 6. (Static route) Decode it by hand

Disassemble `main`:

```bash
objdump -d ctf.exe -M intel
```

`main` does three things:

1. Reads the first two bytes of its own loaded image (`"MZ"`, from the DOS header) and uses them as the first two bytes of a 5-byte key. The key is `4D 5A 5A C3 17`.
2. Decodes a 17-byte blob from `.rdata`, one byte at a time: `out[i] = blob[i] ^ key[i % 5] ^ ((i * 7) & 0xff)`.
3. Calls `puts` on the result.

The decoy string sits behind a comparison that is never true, so it is never printed.

Pull the blob out of `.rdata` and decode it:

```python
blob = bytes([0x7d,0x0e,0x07,0x93,0x48,0x15,0x27,0x5a,0xb5,0x1b,0x54,0x40,0x3e,0xca,0x3e,0x77,0x57])
key  = [0x4d, 0x5a, 0x5a, 0xc3, 0x17]
print(bytes(b ^ key[i % 5] ^ ((i * 7) & 0xff) for i, b in enumerate(blob)).decode())
```

## Flag

```
0SSEC{W1N3_W0RKS}
```

## Takeaways

- A JPEG much larger than its dimensions suggest probably has appended data.
- `strings | tail` and running `unzip` directly on the image are quick ways to catch an appended ZIP.
- A flag you find with `strings` may be a decoy. If the program never prints it, keep going.
- Running the binary is the easy route. Reading the decode routine and reproducing it is the static route.

## Tools

`file`, `exiftool`, `binwalk`, `strings`, `unzip`, `wine`, `objdump`, Python