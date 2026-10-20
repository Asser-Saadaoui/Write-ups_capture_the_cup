# Supernova Challenge – CTF Write-up

**Category:** Forensics / Stego / Crypto\
**Event:** Capture the Cup\
**Flag:** `0SSEC{Y0u_found_th3_0n3_pi3c3}`

## Summary

A single JPEG (`supernova.jpg`, One Piece Supernovas) hides a three-layer chain: JPEG comments with decoy flags and an AES-encrypted blob, then a gzip/tar archive appended to the image, then a final AES-encrypted flag whose key is the SHA-256 of a second image.

```
supernova.jpg
 ├─ JPEG comments  ->  base64 hint  ->  AES blob  ->  3 decoy flags (clues)
 └─ appended gzip  ->  tar  ->  tr4f4lg4r.jpg + flag.enc
                                  flag.enc + sha256(tr4f4lg4r.jpg)  ->  FLAG
```

## Step 1: Metadata

```bash
exiftool -a supernova.jpg
```

Three `Comment` fields:

1. `0SSEC{34sy_huh?}`: fake flag .
2. A base64 string. Decoded, it says: not every flag is real, the fake ones are clues, and the third comment is AES-256-CBC encrypted with key = SHA-256 of the string `"0SSEC"`, base64-encoded, with no IV ("express this mathematically" = IV of zeros).
3. A base64 blob (the ciphertext).

## Step 2: Decrypt the third comment

Key: SHA-256 of `0SSEC`. Use `printf`, not `echo`, because `echo` appends `\n` and changes the hash.

```bash
printf '0SSEC' | sha256sum
# fbaf00e4f2b54bc2280a0f33523e33f6c73529b7e6f41763c400292660dec697
```

Save the blob to `text`, then:

```bash
openssl enc -d -aes-256-cbc -a \
  -K fbaf00e4f2b54bc2280a0f33523e33f6c73529b7e6f41763c400292660dec697 \
  -iv 00000000000000000000000000000000 \
  -in text
```

Output (all decoys, but they are hints):

```
0SSEC{y0u_4re_n0t_cl0s3_3n0ugh_t0_th3_0n3_pi3c3}
0SSEC{l4w_h4s_256_4b1l1t13s}
0SSEC{tr4f4lg4r_is_th3_k3y}
```

- "not close enough to the one piece": the real flag is elsewhere.
- "Law has 256 abilities": SHA-256.
- "Trafalgar is the key": something named `tr4f4lg4r` is the key.

## Step 3: Data appended to the image

```bash
binwalk supernova.jpg
```

```
0        0x0     JPEG image, total size: 41351 bytes
41351    0xA187  gzip compressed data, total size: 63586 bytes
```

The JPEG is 105 KB but its real size is 41351 bytes, so a gzip stream is appended after it.

```bash
binwalk -e supernova.jpg
cd extractions/supernova.jpg.extracted/A187
file decompressed.bin      # POSIX tar archive (GNU)
tar xf decompressed.bin
```

Contents: `tr4f4lg4r.jpg` (63 KB) and `flag.enc` (32 bytes, two AES blocks).

## Step 4: Decrypt the flag

The key is the SHA-256 of the file `tr4f4lg4r.jpg`, as the third decoy suggested. The IV is zeros again.

```bash
sha256sum tr4f4lg4r.jpg
# 00bb3e572a759ff90b039dbafb7ee3bde477282925cb3db77b6ff0c32b2feb6f

K=$(sha256sum tr4f4lg4r.jpg | cut -d' ' -f1)
openssl enc -d -aes-256-cbc -K $K -iv 00000000000000000000000000000000 -in flag.enc
```

```
0SSEC{Y0u_found_th3_0n3_pi3c3}
```


## Takeaways

- Compare a JPEG's real size (from binwalk) with its file size: the difference is appended data.
- Treat decoy flags as clues, not answers.
- Never trust extensions: `decompressed.bin` was a tar archive.
- Raw-key AES with openssl needs `-K` (hex key) and `-iv` (32 hex chars), and a base64 input needs `-a`.

## Tools

`exiftool`, `base64`, `sha256sum`, `openssl enc`, `binwalk`, `file`, `xxd`, `tar`z

