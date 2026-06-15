# mmkc-creator-image2psd-skill
> MMKC project description: Convert images and AI-generated assets into layered PSD files with full-canvas PNG layers, transparency cleanup, color splits, and pure-Python PSD writing.
> MMKC 项目描述：把图片或 AI 生成素材整理为可编辑 PSD 图层，支持全画布 PNG 图层、白底转透明、颜色拆层和纯 Python PSD 写入。


[中文](./README.md) | English

`mmkc-creator-image2psd-skill` is a Codex skill for turning one or more raster images into a layered PSD. It is designed for image-to-PSD workflows such as poster decomposition, product-scene cutouts, AI-generated element assembly, white-background removal, color-cluster splitting, and Photoshop-ready layer export.

The bundled PSD writer is pure Python and does not require Photoshop, ImageMagick, Wand, or `psd-tools`.

## What It Does

- Assemble multiple image or raster text layers into one PSD.
- Preserve layer names and alpha channels.
- Export a flattened PNG preview.
- Export full-canvas transparent PNG layers for manual Photoshop stacking at `(0, 0)`.
- Split a flat image into color-cluster raster layers.
- Create a per-task `projects/YYYYMMDD_slug/` folder for source files, intermediate images, PSD output, previews, and diagnostics.
- In Codex, pair naturally with the `imagegen` skill for generating, editing, or rebuilding image elements before PSD assembly.

## Install

Copy this folder into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
cp -R mmkc-creator-image2psd-skill ~/.codex/skills/
```

Or clone the whole this repository and copy/symlink the skill:

```bash
git clone <your-github-url>/mmkc-creator-image2psd-skill.git
mkdir -p ~/.codex/skills
ln -s "$PWD/mmkc-creator-image2psd-skill" ~/.codex/skills/mmkc-creator-image2psd-skill
```

Install runtime dependencies:

```bash
python3 -m pip install -r ~/.codex/skills/mmkc-creator-image2psd-skill/scripts/requirements.txt
```

Required dependencies are `Pillow` and `numpy`. `opencv-python` improves subject masks and light foreground preservation. `scikit-learn` improves `split-colors --method kmeans`.

## Quick Start In Codex

Ask Codex something like:

```text
Use mmkc-creator-image2psd-skill to turn this image into a PSD.
Keep element positions unchanged, split the main objects and text into separate layers,
and put all process images under the skill project's projects folder.
```

For Codex image generation workflows:

```text
Use imagegen to generate separate background, subject, title, and decoration images,
then use mmkc-creator-image2psd-skill to assemble them into a PSD.
```

## Command-Line Usage

Initialize a project folder:

```bash
python3 mmkc-creator-image2psd-skill/scripts/init_project.py lifestyle_product \
  --source input.png
```

Assemble layers from a manifest:

```bash
python3 mmkc-creator-image2psd-skill/scripts/image2psd.py assemble \
  --manifest mmkc-creator-image2psd-skill/projects/20260503_lifestyle_product/manifest.json
```

Assemble positional images directly:

```bash
python3 mmkc-creator-image2psd-skill/scripts/image2psd.py assemble bg.png logo.png title.png \
  --first-is-background \
  --names "Background,Logo,Title" \
  --output output.psd \
  --save-layers layers
```

Split one flat image into color layers:

```bash
python3 mmkc-creator-image2psd-skill/scripts/image2psd.py split-colors poster.png \
  --output poster-color-layers.psd \
  --num-colors 10 \
  --ignore-color white \
  --save-layers poster-color-layers
```

## Manifest Example

Layer order is bottom-to-top.

```json
{
  "canvas": {
    "width": 1200,
    "height": 1600,
    "composite_background": "#ffffff"
  },
  "output": "output.psd",
  "preview": "output.preview.png",
  "save_layers_dir": "psd_full_canvas_layers",
  "zip_layers": "psd_full_canvas_layers.zip",
  "layers": [
    {
      "name": "Background",
      "file": "layer_sources/background.png",
      "fit": "cover",
      "remove_background": "none"
    },
    {
      "name": "Subject",
      "file": "layer_sources/subject.png",
      "remove_background": "white-preserve"
    },
    {
      "name": "Title",
      "type": "text",
      "text": "Event Title",
      "x": 80,
      "y": 120,
      "font_size": 76,
      "color": "#1c1712",
      "max_width": 960
    }
  ]
}
```

## Background Removal Modes

- `none`: keep the image as-is.
- `white`: convert white background to alpha.
- `white-preserve`: white-to-alpha plus a soft structure mask for pale foregrounds.
- `corner`: sample the four corners as the background color.
- `color`: use an explicit `color` field in the layer spec.

## Project Output Layout

Each real task should live under:

```text
mmkc-creator-image2psd-skill/projects/YYYYMMDD_slug/
├── original_reference.png
├── manifest.json
├── layer_sources/
├── psd_full_canvas_layers/
├── imagegen_assets/
├── diagnostics/
├── output.psd
├── output.preview.png
├── psd_full_canvas_layers.zip
└── process_notes.md
```

Generated project outputs are ignored by Git by default. Keep only `.gitkeep` in `projects/`.

## Notes

- Text layers created by the manifest are raster layers, not editable Photoshop text objects.
- Semantic decomposition from a single flat image is inherently approximate. For exact editability, generate or provide separate source elements whenever possible.
- If a user asks to preserve exact relative position, use full-canvas transparent PNG layers so Photoshop can stack every layer at `(0, 0)`.

## License

MIT
