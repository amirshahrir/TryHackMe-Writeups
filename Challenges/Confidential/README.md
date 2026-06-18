
```
Room         : Confidential
Platform     : TryHackMe
Difficulty   : Easy
Date         : 18/06/2026
Author       : Amir Shahrir
Tools Used   : pdfimages, zbarimg (attempted), Linux Terminal
Status       : Completed
```

---

### Executive Summary

The challenge presented a PDF document containing a QR code concealed beneath an overlaid image element. Visual inspection of the file confirmed the QR code was present but inaccessible through normal viewing. The embedded images were enumerated and extracted individually using pdfimages, isolating the QR code as a standalone PNG file. A command-line decoding tool (zbarimg) was unavailable in the lab environment and could not be installed, so a smartphone camera was used as an alternative — successfully decoding the QR code and recovering the flag.

---

### Objectives

- Locate and examine the provided PDF file within the lab environment
- Identify and enumerate image layers embedded within the document
- Extract and isolate the QR code from the obscuring overlay
- Decode the QR code to recover the hidden flag

---

### Environment & Tools

| Component         | Details                                                        |
| ----------------- | -------------------------------------------------------------- |
| Platform          | TryHackMe in-browser VM (Ubuntu)                               |
| Working Directory | `/home/ubuntu/confidential`                                    |
| Tools             | pdfimages, zbarimg (attempted), Linux Terminal |

---

### Methodology

The working directory `/home/ubuntu/confidential` was navigated to, where the target file `Repdf.pdf` was located. Opening the PDF through the desktop viewer confirmed a QR code was visible within the document, but an overlaid image element was blocking it — making the code unreadable through normal means.

To understand the document's internal structure, `pdfimages` was run with the `-list` flag to enumerate all embedded image objects without extracting them:

```bash
pdfimages -list Repdf.pdf
```

The output revealed three image objects: one classified as smask (a soft mask, typically used as a transparency or alpha channel layer) and two standard image types. This confirmed that multiple image layers were stacked within the PDF — one of which was the obscuring overlay, and another was the underlying QR code.
All images were then extracted as PNG files using the following command:

```bash
pdfimages -png Repdf.pdf extracted_qr
```

PNG was chosen deliberately — it is a lossless format, meaning no image data is discarded during conversion. For machine-readable content such as QR codes, any compression artefacts introduced by a lossy format (such as JPEG) risk corrupting the encoded data and causing decoding failures.
This produced a set of sequentially named files (`extracted_qr-000.png, extracted_qr-001.png,` etc.). Cross-referencing the extraction order with the desktop viewer confirmed that extracted_qr-000.png corresponded to the QR code image.

An attempt was made to decode the QR code programmatically using zbarimg:
bashzbarimg extracted_qr-000.png

```bash
zbarimg extracted_qr-000.png
```

The tool was not available on the lab VM, and the restricted environment prevented installation of new packages. As a practical workaround, the extracted PNG was displayed on screen and scanned using a smartphone camera, which decoded the QR code and returned the flag: [REDACTED].

---

### Lessons Learned

- PDF documents can contain layered image objects, where one element obscures another. Running `pdfimages -list` before extracting anything is a useful first step — it reveals how many image objects exist, what types they are, and informs which ones are worth investigating. Jumping straight to extraction without listing first means working blind.

- PNG is the correct output format when the extracted image needs to be machine-readable. Lossless extraction preserves every pixel exactly as stored in the document. A lossy format like JPEG introduces compression artefacts that can corrupt QR code patterns and cause decoding failures.

- `zbarimg` is a command-line QR and barcode decoder that is not installed by default on most Linux distributions. It is part of the `zbar-tools` package. Knowing its name and purpose is useful; knowing it may be absent in a restricted lab environment is equally useful.

- When a preferred tool is unavailable, alternative methods can substitute without compromising the result. Using a smartphone camera to decode a QR code from a screen is unconventional, but effective — and a practical reminder that problem-solving in security work is not always linear.

---

_Write-up by Amir Shahrir | https://github.com/amirshahrir | Completed: [18/06/2026]_
_Note: This write-up is for educational purposes. All activities were conducted in an isolated, legal lab environment provided by TryHackMe._

```

```
