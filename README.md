# Capture The Cup — Forensics Challenges

A collection of beginner-friendly digital forensics challenges authored for **Capture The Cup**, organized by **OSSEC (Open Source Software ENSI Club)** at the National School of Computer Science (ENSI), Manouba, Tunisia, on **18 October 2026**.

The challenges were designed to introduce participants to practical digital forensics, steganography, data extraction, and basic reverse engineering through approachable, hands-on investigations.

## Challenges

| Challenge | Category | Difficulty | Main Techniques |
|---|---|---|---|
| Supernova | Forensics / Steganography / Cryptography | Beginner-friendly | JPEG metadata, appended archives, AES-256-CBC, SHA-256 |
| Vintage | Forensics / Steganography / Reverse Engineering | Beginner-friendly | Appended ZIP, PE analysis, static reverse engineering, XOR |
| Welcome to CC | Forensics | Beginner | String extraction, plain-text flag discovery |
| Haramball | Forensics | Beginner | JPEG comments, metadata analysis, duplicate COM segments |
| Haramball 2 | Forensics | Beginner | Decoy flags, duplicate metadata, JPEG structure |
| Feel It | Forensics / Audio | Beginner-friendly | WAV sample analysis, signal encoding, hexadecimal decoding |
| Dah Dah Challenge | Forensics / Steganography | Beginner-friendly | Appended ZIP, nested archives, spectrogram analysis, password-protected ZIP |

*Difficulty labels describe the intended learning experience, rather than an independently measured difficulty rating.*

## Learning Objectives

These challenges introduce participants to:

- Inspecting file metadata and identifying hidden information.
- Detecting appended data and extracting embedded archives.
- Understanding JPEG structure and duplicate comment segments.
- Analyzing audio samples and visualizing spectrograms.
- Recognizing hexadecimal, Base64, and XOR-based encoding.
- Using hashes as cryptographic keys and decrypting AES-encrypted data.
- Examining Windows executables through static analysis.
- Distinguishing decoy flags from the intended solution.

## Tools

Depending on the challenge, the investigation may involve:

- `file`
- `strings`
- `exiftool`
- `binwalk`
- `xxd` and `hexdump`
- `base64`
- `openssl`
- `unzip` and `tar`
- `sha256sum`
- `objdump`
- Python
- `scipy` and `matplotlib`
- Wine

## Repository Structure

Each challenge includes its available challenge artifacts and a write-up documenting the intended solution.

```text
.
├── README.md
├── Supernova/
│   ├── README.md
│   ├── supernova.jpg
│   └── solution/
├── Vintage/
│   ├── README.md
│   ├── wine_challenge.jpg
│   └── solution/
├── Welcome-to-CC/
├── Haramball/
├── Feel-It/
└── Dah-Dah-Challenge/
```

The tree above is illustrative; the actual folders and filenames may differ.

## About the Authoring

The challenges were designed around recognizable digital artifacts and layered clues, encouraging participants to investigate beyond a file's visible contents. Some focus on a single forensic discovery, while others combine several techniques into a multi-step investigation.

The goal was to make digital forensics approachable for beginners while encouraging systematic analysis, curiosity, and attention to detail.

## Event

- **Event:** Capture The Cup
- **Organizer:** OSSEC — Open Source Software ENSI Club
- **Venue:** ENSI, Manouba, Tunisia
- **Date:** 18 October 2026

## Disclaimer

This repository is intended for cybersecurity education, CTF practice, and reproducible challenge analysis. Challenge files and solutions are shared for educational purposes, subject to any applicable distribution permissions.

---

*Created for Capture The Cup at ENSI — October 2026.*
