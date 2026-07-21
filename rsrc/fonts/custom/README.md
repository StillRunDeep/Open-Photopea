# Bundled fonts

This directory contains the small bundled font set used by the text tool.

## Included fonts

| Path | Font | Purpose | License | Source |
| --- | --- | --- | --- | --- |
| `noto-sans-sc/0` | Noto Sans CJK SC Regular | Simplified Chinese sans-serif | SIL Open Font License 1.1 | https://github.com/notofonts/noto-cjk |
| `noto-sans-sc/1` | Noto Sans CJK SC Bold | Simplified Chinese sans-serif | SIL Open Font License 1.1 | https://github.com/notofonts/noto-cjk |
| `noto-serif-sc/0` | Noto Serif CJK SC Regular | Simplified Chinese serif | SIL Open Font License 1.1 | https://github.com/notofonts/noto-cjk |
| `noto-serif-sc/1` | Noto Serif CJK SC Bold | Simplified Chinese serif | SIL Open Font License 1.1 | https://github.com/notofonts/noto-cjk |

The English fonts used by common text layers are stored under the original catalog paths already referenced by `code/FNTS.js`:

| Path | Font | License | Source |
| --- | --- | --- | --- |
| `../fs/roboto-2014/4`, `5`, `8`, `9` | Roboto Regular / Italic / Bold / BoldItalic | Apache License 2.0 / Google Fonts distribution | https://github.com/googlefonts/roboto-2 |
| `../fs/liberation-sans/*` | Liberation Sans | SIL Open Font License 1.1 | https://github.com/liberationfonts/liberation-fonts |
| `../fs/liberation-serif/*` | Liberation Serif | SIL Open Font License 1.1 | https://github.com/liberationfonts/liberation-fonts |
| `../fs/liberation-mono/*` | Liberation Mono | SIL Open Font License 1.1 | https://github.com/liberationfonts/liberation-fonts |

## Adding more fonts

1. Put the font binary below `rsrc/fonts/`.
2. Add or update the matching entry in `code/FNTS.js`.
3. Make sure the PostScript name in `code/FNTS.js` matches the font's internal PostScript name.
4. If the font should be used as a replacement for PSD text layers, add a mapping in `rr.prototype.Dx` inside `code/pp.js`.
