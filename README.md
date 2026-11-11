# image-specs

Open JSON data for working with images on the web: **platform image sizes with official sources**, **upload size limits**, **image format properties**, and **measured conversion and compression results**.

Every record either quotes an official source (page URL, word-for-word quote, date checked) or comes from a test run with [Pixohub](https://pixohub.com)'s in-browser tools. Each record links to the Pixohub page that performs the change (resize, compress, convert).

| File | Contents | Records |
|---|---|---|
| [`data/platform-image-sizes.json`](data/platform-image-sizes.json) | Official dimensions, aspect ratios, upload limits, cropping and safe zones for social, video, marketplace and print placements | 93 placements (53 with an official dimension record) |
| [`data/upload-size-limits.json`](data/upload-size-limits.json) | File-size and dimension limits for email attachments, exam and visa forms, avatars and marketplaces | 83 |
| [`data/image-formats.json`](data/image-formats.json) | JPG, PNG, WebP, AVIF, HEIC, SVG, GIF, BMP, TIFF, ICO, PSD, TGA, PPM: compression, colour depth, transparency, animation, metadata, support | 13 |
| [`data/conversion-tests.json`](data/conversion-tests.json) | Format-to-format conversion measured on a photo and a transparent graphic: size change, SSIM/PSNR, alpha and EXIF survival | 108 pairs |
| [`data/compression-tests.json`](data/compression-tests.json) | Compress-to-size (10 KB – 5 MB) on 20 phone photos and the Kodak suite: hit rate, median size, resolution kept, SSIM | 23 targets |
| [`data/pixohub-resize-presets.json`](data/pixohub-resize-presets.json) | The sizes Pixohub's resize presets produce (working values, **not** official specs) | 143 |

The same files are served with open CORS at `https://pixohub.com/open-data/<file>.json` and documented at [pixohub.com/developers](https://pixohub.com/developers).

## Record shapes

### platform-image-sizes.json

```jsonc
{
  "id": "youtube-thumbnail",
  "platform": "YouTube",
  "usage": "Thumbnail",
  "official": {                       // null when no official dimension was found
    "recommended": { "w": 3840, "h": 2160 },
    "min": { "w": 640 },
    "aspect_ratio": "16:9",
    "max_file_size": "2 MB (mobile upload) / 50MB (desktop upload)",
    "formats": ["JPG", "PNG"],
    "source": { "url": "https://support.google.com/youtube/answer/72431", "publisher": "YouTube Help (Google)", "quote": "…", "checked": "2026-11-08" }
  },
  "facts": [                          // processing, cropping, safeZone, uploadLimits
    { "type": "cropping", "text": "…", "source": { "url": "…", "quote": "…", "checked": "2026-10-24" } }
  ],
  "pixohub_preset": { "width": 1280, "height": 720, "aspect": "16:9", "matches_official": "partial" },
  "notes": "…",
  "tool": "https://pixohub.com/resize/youtube-thumbnail"
}
```

`matches_official` is `true`, `"partial"` (same ratio or inside a documented range, different pixels), `false`, or `null` (no official pixel value to compare).

### upload-size-limits.json

`organization`, `what`, `kind` (email, government-exam, visa-passport, community-avatar, social, ecommerce, …), `min_kb`, `max_kb`, `other_requirements`, `source {name, url, quote, checked}`, `tool` (the largest Pixohub compress-to-size page that fits).

## Usage

```js
const res = await fetch("https://pixohub.com/open-data/platform-image-sizes.json");
const { items } = await res.json();
const story = items.find((i) => i.id === "instagram-story");
```

```python
import json, urllib.request
data = json.load(urllib.request.urlopen("https://pixohub.com/open-data/upload-size-limits.json"))
print([l for l in data["items"] if "Gmail" in l["organization"]])
```

## MCP server

[![Pixohub image specs MCP server](https://mcprush.com/badge/pixohub-image-specs-mcp/card.svg)](https://mcprush.com/to/pixohub-image-specs-mcp?ref=badge)

The same data is queryable by AI assistants through a free, read-only [MCP](https://modelcontextprotocol.io) server (Streamable HTTP, no key):

```
https://mcp.pixohub.com/mcp
```

```bash
claude mcp add --transport http pixohub https://mcp.pixohub.com/mcp
```

Tools: `get_platform_image_size`, `find_upload_size_limit`, `recommend_image_format`, `compression_advice`, `conversion_test_result`.

## Method and caveats

- **Official sources only.** Help centres, developer docs, ads guides and standards text. Third-party "size guide" blogs are not used. Where an official page could not be read (blocked or client-rendered), the field is `null` rather than filled from elsewhere. Paper sizes whose only readable source was Wikipedia are left out of `official`.
- **Platforms change specs.** Each fact carries the date it was checked; re-check the linked page before relying on a hard limit.
- **Measurements** come from Pixohub's in-browser pipeline (Chromium via Playwright). Test photos are from Wikimedia Commons (licences listed in `compression-tests.json`) and the Kodak Lossless True Color Image Suite. Method: [pixohub.com/editorial-policy](https://pixohub.com/editorial-policy#testing).
- Found an outdated value? Open an issue with the official link.

## Licence

Compilation and measurements: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — credit "Pixohub (pixohub.com)" with a link. Quoted passages remain the property of their publishers and are included only to show where each fact comes from.
