---
name: webp-converter
description: This skill should be used when the user asks to convert images to webp format, including requests like "转成webp", "convert to webp", "图片转webp", "compress images to webp", or batch conversion of images.
version: 1.0.0
---

# WebP Image Converter

Convert images to WebP format using `cwebp` (WebP encoder tool).

## Overview

This skill provides guidance for converting common image formats (PNG, JPEG, TIFF, BMP) to WebP format, supporting both single-file and batch conversion with customizable quality settings.

## Available Tool

`cwebp` is the WebP encoder. It's pre-installed at `/opt/homebrew/bin/cwebp`.

If it's not available, install with: `brew install webp`

## Basic Usage

### Single File Conversion

```bash
cwebp input.png -o output.webp
```

### With Quality Control

```bash
cwebp -q 80 input.jpg -o output.webp
```

Quality range: 0 (smallest) to 100 (best), default is 75.

### Presets

Use `-preset` to optimize for the image type (must be placed first):

```bash
cwebp -preset photo -q 80 input.jpg -o output.webp      # for photos
cwebp -preset drawing -q 80 input.png -o output.webp    # for drawings/illustrations
cwebp -preset icon -q 80 input.png -o output.webp       # for icons
cwebp -preset picture -q 80 input.png -o output.webp    # for general pictures
cwebp -preset text -q 80 input.png -o output.webp       # for text-heavy images
```

### Lossless Mode

```bash
cwebp -lossless input.png -o output.webp
```

## Batch Conversion

Convert all images in a directory:

```bash
for f in *.png; do cwebp "$f" -o "${f%.png}.webp"; done
for f in *.jpg; do cwebp "$f" -o "${f%.jpg}.webp"; done
```

Mixed formats:

```bash
for f in *.{png,jpg,jpeg,tiff,bmp}; do
  [ -f "$f" ] && cwebp "$f" -o "${f%.*}.webp"
done
```

## Key Options Reference

| Option | Description | Default |
|--------|-------------|---------|
| `-q <float>` | Quality (0-100) | 75 |
| `-preset <string>` | Preset mode (photo/picture/drawing/icon/text) | default |
| `-lossless` | Lossless encoding | off |
| `-m <int>` | Compression method (0=fast, 6=slowest) | 4 |
| `-mt` | Multi-threading | off |
| `-resize <w> <h>` | Resize before encoding | none |
| `-noalpha` | Discard alpha channel | off |
| `-z <int>` | Lossless compression level (0-9) | - |

## Workflow

1. Identify the source image(s) the user wants to convert
2. Determine appropriate quality/preset based on context:
   - Photos: `-preset photo -q 80`
   - Screenshots/UI: `-preset picture -q 85`
   - Icons/logos: `-preset icon -q 90`
3. Run `cwebp` to convert
4. Note the compression ratio in the output and inform the user
5. If the user wants to replace originals, suggest using `-o` with the same basename
