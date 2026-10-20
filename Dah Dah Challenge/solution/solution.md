# CTF Writeup: dah_dah_challenge

**Flag:** `0SSEC{D0NT_B3_P4TRICK}`

## Challenge

A single file, `chall.png`, was provided — a SpongeBob/Patrick image. The challenge was built to be "solved only with binwalk," meaning the intended path is: find hidden data via binwalk's signature scanning, not by eyeballing or guessing.

## Layer 1 — PNG with an appended archive·

`file chall.png` reports a valid PNG (447x498). But the PNG's `IEND` chunk ends at offset `123280`, while the file itself is about 1.9 MB — far larger than a clean PNG of that size should be.

Looking right after `IEND`, there's a `PK\x03\x04` signature at offset `123288`: a zip archive has been appended to the image. This is exactly the kind of thing `binwalk` is built to find — it scans for file signatures regardless of where they sit in the file, rather than assuming one file = one format.

```sh
binwalk chall.png      # flags PNG + Zip at offset 123288
binwalk -e chall.png   # extracts it to _chall.png.extracted/
```

**Note:** this outer zip was not password-protected, so `unzip chall.png` or even `strings chall.png` also recovers it directly — a shortcut around the intended binwalk-only path. (Flagged this to the challenge author separately.)

## Layer 2 — Nested archives

Extracting the appended zip gives a `hidden/` directory with two files:

- `hidden/dahdah.zip` (227 bytes, password-protected, contains one file: `flag.txt`)
- `hidden/patrick_hidden.wav` (~2.2 MB audio)

So the flag itself is encrypted inside `dahdah.zip`, and the password has to come from somewhere — presumably the WAV file.

## Layer 3 — Audio steganography (spectrogram text)

`patrick_hidden.wav` is a ~25 second mono 44.1kHz PCM file. The first 3 seconds are silent; LSB analysis of the sample data showed nothing meaningful, ruling out simple LSB steganography.

Rendering a spectrogram (`matplotlib.specgram`, NFFT=1024) of the audio revealed hand-drawn text embedded in the 8–15 kHz frequency band:

```
P4SSWORD_0R_IDK
```

This is a classic technique: drawing text or an image directly into the frequency domain, visible only when you look at the spectrogram rather than listening to the audio.

**Gotcha:** the drawn `0` (zero) in "0R" is visually near-identical to a letter `O` at spectrogram resolution. Reading it as `P4SSWORD_OR_IDK` (letter O) consistently failed to decrypt the zip. A small brute force over leet-style substitutions of the drawn word (`O`/`0`, `A`/`4`, etc.) across the separators recovered the correct password: **`P4SSWORD_0R_IDK`** (digit zero).

This subtlety doubles as a thematic hint — the final flag content (`D0NT_B3_P4TRICK`, i.e. "don't be tricked") is literally about not trusting what looks right at first glance.

## Layer 4 — Decrypting the inner zip

With the correct password, the inner archive opens cleanly:

```python
import zipfile, base64
z = zipfile.ZipFile('dahdah.zip')
data = z.read('flag.txt', pwd=b'P4SSWORD_0R_IDK')
print(base64.b64decode(data).decode())
```

`flag.txt` contained a base64 string, which decodes to the final flag:

```
0SSEC{D0NT_B3_P4TRICK}
```

## Summary of the chain

1. `chall.png` → carve appended zip at offset `123288` (binwalk / unzip)
2. → `hidden/dahdah.zip` (encrypted) + `hidden/patrick_hidden.wav`
3. → spectrogram of WAV reveals password `P4SSWORD_0R_IDK` (note the digit zero, not letter O)
4. → decrypt `dahdah.zip` → `flag.txt` (base64) → decode → `0SSEC{D0NT_B3_P4TRICK}`

## Notes for the challenge author

- The outer zip (step 1) is unencrypted, so `unzip` or `strings` bypasses the intended binwalk-only path. Encrypting it (`zip -r -e`) or switching to a format `unzip` can't parse directly would enforce the binwalk requirement.
- The inner zip uses legacy ZipCrypto (crackable via known-plaintext with `bkcrack` given ~12 bytes of known content, though the 6-byte `0SSEC{` prefix alone wasn't enough here). Switching to AES-256 zip encryption (`zip -e -Z aes` or `7z -mhe=on`) would close that avenue for future iterations.