# Lab 3 — Data Carving with XXD, Binwalk and Scalpe

## Author

**Athanasius Orikeze Alekwe**  
Student ID: **2025/FWSD/11230**  
Course: **Computer and Digital Forensics**  
Instructor: **Aminu Idris**

## Overview

This repository documents **Computer and Digital Forensics Lab 3**, focused on data carving, file-signature analysis, deleted-file recovery, embedded-content extraction, and forensic evidence verification.

The practical demonstrates how useful evidence can still be recovered when normal file-system metadata is missing, damaged, deleted, or unreliable. The investigation used hexadecimal analysis, The Sleuth Kit, Binwalk, Scalpel, and cryptographic hashing to examine and recover data from authorised training evidence.

## Objectives

The laboratory demonstrates how to:

- Examine binary files using `xxd`.
- Identify JPEG Start of Image (`FF D8`) and End of Image (`FF D9`) signatures.
- Create and reverse a hexadecimal dump.
- Verify reconstructed files using MD5 and SHA-256.
- Examine readable metadata-related information using `strings`.
- Analyse forensic images using The Sleuth Kit.
- Inspect partitions, file systems, metadata addresses, sectors, and deleted files.
- Recover deleted files using `icat` and `blkcat`.
- Detect embedded file structures using Binwalk.
- Perform manual embedded-file extraction using offsets.
- Recover deleted JPEG files using Scalpel.
- Verify recovered artefacts using cryptographic hashes.
- Package supporting forensic evidence in a structured and repeatable manner.

## Tools Used

- Kali Linux
- VMware Workstation
- `wget`
- `file`
- `xxd`
- `strings`
- `grep`
- `md5sum`
- `sha256sum`
- `cmp`
- The Sleuth Kit:
  - `img_stat`
  - `mmls`
  - `fsstat`
  - `fls`
  - `istat`
  - `icat`
  - `blkcat`
- `7z`
- Binwalk
- `dd`
- `unzip`
- Scalpel
- Kali Image Viewer
- `zip`

## Laboratory Workflow

### 1. Evidence Preparation

A dedicated forensic workspace was created to separate original evidence, working files, hashes, TSK output, Binwalk output, Scalpel output, screenshots, and notes.

The original evidence files were hashed using MD5 and SHA-256 and then made read-only before analysis.

### 2. JPEG Signature Analysis

`J_ub_law.jpg` was examined with `xxd`.

The analysis confirmed:

- JPEG SOI signature: `FF D8`
- APP1 marker: `FF E1`
- Embedded EXIF structure
- JPEG EOI signature: `FF D9`

A plain hexadecimal dump was created and reversed back into a JPEG file. The reconstructed image produced identical MD5 and SHA-256 values to the source working copy, while `cmp` returned exit status `0`, confirming a byte-for-byte match.

## Metadata Examination

Readable metadata-related strings were recovered from the JPEG using `strings`, `grep`, and `xxd`.

Notable findings included:

- Camera manufacturer: **NIKON CORPORATION**
- Camera model: **NIKON D4**
- Software: **Adobe Photoshop 21.1 (Macintosh)**
- Name-related string: **HOWARD KORN**
- Lens-related string: **17.0-35.0 mm f/2.8**
- Date/time: **2020:08:20 10:20:05**
- Additional date/time: **2013:10:08 18:56:21**
- Adobe/XMP metadata
- Embedded camera-profile information

## The Sleuth Kit Analysis

`Ch01InChap01.dd` was examined using The Sleuth Kit.

`img_stat` identified it as a raw **1,474,560-byte** image with **512-byte sectors**.

`mmls` returned no partition table, so no arbitrary partition offset was used. `fsstat` identified a **FAT12** filesystem.

Deleted entries identified with `fls` included:

- `Billing Letter.doc`
- `confirmation.txt`
- `letter1.txt`
- `Regrets.doc`

A deleted `Billing Letter.doc` at metadata address **8** was investigated using `istat`.

It occupied sectors `237–283` with a total size of **24,064 bytes**.

The file was recovered using `icat`. The same 47 sectors were then extracted directly using `blkcat`.

Sector 237 began with:

```text
D0 CF 11 E0 A1 B1 1A E1
```

The `icat` and `blkcat` recoveries produced identical MD5 and SHA-256 values.

## Binwalk Analysis

The instructor-referenced `File_carving.docx` was not available in the supplied files or inside `120M.7z`.

This limitation was documented, and the Binwalk exercise continued using the other authorised laboratory evidence as permitted by the practical instructions.

Binwalk identified several structures within the evidence, including JPEG data, EXIF/TIFF information, ZIP/Office Open XML structures, XML content, HTML content, PDF signatures, and audio data.

A Microsoft Office Open XML structure was identified beginning at decimal offset **421888**.

The calculated extraction size was **97,702 bytes**, and the object was manually recovered with `dd`.

`file` identified it as:

```text
Microsoft Word 2007+
```

`unzip -t` reported no errors, while the archive contained Word OOXML components including:

```text
[Content_Types].xml
_rels/.rels
word/document.xml
word/media/image1.jpeg
word/styles.xml
docProps/core.xml
docProps/app.xml
```

The recovered object was subsequently hashed using MD5 and SHA-256.

## USB Forensic Image

The `120M.7z` archive produced:

```text
usb_fat_carving.001
usb_fat_carving.001.txt
```

The forensic image had the following properties:

```text
Image size: 124,780,544 bytes
Image type: Raw
Sector size: 512 bytes
```

MD5:

```text
ba4a1d0ba49f4a6667b00a3b3e85e604
```

SHA-256:

```text
9bfe4b5634ade30764f0e581a4686930f7fab0472378595b789b0cdb248c91d9
```

`mmls` confirmed the FAT16 partition began at sector **128**.

The FAT16 filesystem was further identified as:

```text
OEM: MSDOS5.0
Volume Label: USB
Sector Size: 512 bytes
Cluster Size: 2048 bytes
Sectors before filesystem: 128
```

## JPEG Carving with Scalpel

A working copy of the Scalpel configuration was created and the JPEG rules for both **EXIF** and **JFIF** images were enabled.

Scalpel successfully carved **17 JPEG files** from the USB forensic image.

For every carved JPEG:

- the file type was checked,
- MD5 hashes were generated,
- SHA-256 hashes were generated,
- carving information was recorded in `audit.txt`.

Several carved files were successfully opened in the Kali image viewer, including:

```text
00000009.jpg
00000010.jpg
00000011.jpg
```

confirming that usable JPEG content had been recovered.

## Evidence Packaging

The final supporting evidence was organised into a clean submission structure containing:

```text
hashes/
notes/
tsk/
binwalk/
scalpel/
other_recovered/
```

A SHA-256 manifest was created for the supporting evidence before the directory was compressed into the final evidence ZIP.

The completed ZIP archive was tested to ensure that the compressed files could be read successfully.

## Key Findings

- JPEG signatures and EXIF information were successfully identified using hexadecimal analysis.
- A JPEG reconstructed from its hexadecimal dump was byte-for-byte identical to the source working copy.
- Camera, software, date, lens, and other metadata-related information was recovered.
- A deleted Microsoft Word document was recovered using both logical metadata-based and direct sector-level methods.
- The FAT16 partition of the USB forensic image was confirmed to begin at sector **128**.
- Binwalk identified multiple embedded structures within the USB image.
- A valid Microsoft Word OOXML structure was manually extracted using Binwalk-derived offsets.
- Scalpel successfully recovered **17 JPEG artefacts**.
- Multiple carved JPEGs opened correctly after recovery.
- Cryptographic hashes were used throughout the investigation to support integrity verification.

## Limitation

The instructor-referenced `File_carving.docx` was not available among the supplied evidence and was also not present inside `120M.7z`. The required DOCX-specific workflow could therefore not be reproduced directly.

In accordance with the laboratory instructions, the Binwalk exercise was instead completed using the other authorised evidence, including the supplied USB forensic image.

## Conclusion

This laboratory demonstrated how forensic recovery can continue even where files have been deleted or normal file-system information cannot be relied upon. By combining signature analysis, hexadecimal inspection, metadata examination, file-system analysis, direct sector extraction, embedded-file discovery, and file carving, useful evidence was successfully identified and recovered.

The investigation also maintained evidence integrity by separating original evidence from working copies and repeatedly using MD5 and SHA-256 hashes to verify recovered and reconstructed artefacts.



---

**Repository purpose:** Academic digital-forensics laboratory documentation using authorised training evidence only.
