<img src="assets/logo.svg" width="56" height="56" alt="LabelSketch logo" align="left">

# LabelSketch

Draw boxes & polygons, tag classes, export JSON.

<br clear="left">

LabelSketch is a single-file, static HTML tool for labelling images as training input for ANN models. It has no backend, no build step, no API keys, and no CDN dependencies — everything runs client-side in the browser, fully offline, and output is a plain JSON file you can load into a PyTorch/TensorFlow `Dataset`, convert to COCO/YOLO, or feed into any custom training pipeline.

## Features

- **Import images**: drag & drop or file picker, any number of images, nothing uploaded anywhere
- **Two annotation shapes**: bounding box (for detection) and polygon (for segmentation)
- **Whole-image tags**: for classification-style datasets that don't need regions, just labels per image
- **Class manager**: add/remove classes, each auto-assigned a color; pick the active class before drawing
- **Annotation list**: every shape on the current image shown in the sidebar, color-coded; click to select and re-tag, click ✕ to delete
- **Edit after drawing**: with the select tool, drag a shape's body to move it, drag a corner/vertex handle to resize a box or reshape a polygon, double-click a polygon vertex to remove it
- **Zoom, all by mouse**: scroll/trackpad zooms toward the cursor, fit/100%/±zoom buttons, or pick the dedicated **zoom tool** and click the image (click again on the tool to flip it to zoom-out, or hold <kbd>Alt</kbd>/right-click for a one-off zoom-out) — no keyboard required
- **Pan**: with the select tool, drag empty canvas space (or drag the scrollbars) once you're zoomed in
- **Right-click a shape** to relabel it from a small class menu, or delete it, without leaving the canvas
- **Import / export JSON**: export writes every image's tags + annotations (pixel and normalized 0–1 coordinates); import restores annotations onto matching filenames, or reconstructs images entirely if the export embedded image data
- **Export Dataset (ZIP)**: a training-ready package — every image renamed to a content hash (independent of whatever filename it arrived with), one label file per image, and a manifest tying it together — see [Structured dataset export](#structured-dataset-export) below

## Usage

1. **Import Images** (or drag & drop) to load one or more images.
2. In the **Classes** panel, add a class name (e.g. `cat`, `defect`, `person`) — it becomes the active class.
3. Pick a tool: **bounding box** (drag a rectangle) or **polygon** (click points, click the first point or press <kbd>Enter</kbd> to close, <kbd>Esc</kbd> to cancel).
4. Repeat for as many regions/classes as needed. Click a shape (on the canvas or in the **Annotations** list) to select it — re-tag its class in the sidebar, drag its body to move it, drag a handle to resize/reshape it, or delete it. Or just **right-click any shape** to pick a new class (or delete it) from a popup menu, no sidebar needed.
5. For classification-only labelling, skip drawing and just add **Image tags** at the bottom of the sidebar.
6. Click **Export Labels JSON** to download everything. Check **embed images** first if you want the JSON to be fully self-contained (larger file, but portable without the original image files).
7. **Import Labels JSON** re-applies a previous export: annotations attach to already-loaded images with matching filenames, or — if the export embedded image data — the images themselves are recreated.

### Keyboard shortcuts

`V` select · `B` bounding box · `P` polygon · `Z` zoom tool · `Delete`/`Backspace` remove selected annotation · `Esc` cancel current draw / deselect / close menu · `+`/`-` zoom in/out · `0` fit to window

## Output format

```json
{
  "format": "labelsketch-v1",
  "exported_at": "2026-08-14T12:00:00.000Z",
  "classes": [{ "name": "cat", "color": "#5aa9e6" }],
  "images": [
    {
      "filename": "photo.jpg",
      "width": 4032,
      "height": 3024,
      "tags": ["outdoor"],
      "annotations": [
        {
          "class": "cat",
          "color": "#5aa9e6",
          "shape_type": "bbox",
          "bbox": { "x": 812, "y": 400, "width": 620, "height": 540 },
          "bbox_normalized": { "x": 0.2014, "y": 0.1323, "width": 0.1538, "height": 0.1786 }
        }
      ]
    }
  ]
}
```

- Coordinates are given both in original-image pixels and normalized 0–1 (handy for YOLO-style `cx,cy,w,h` or any resolution-independent pipeline).
- Polygons carry `points` / `points_normalized` (arrays of `[x, y]`) instead of `bbox`.
- `image_data` (a data URL) is only present per-image when **embed images** was checked at export time.

## Structured dataset export

**Export Dataset (ZIP)** packages the whole session into one archive, organized for a training pipeline rather than for re-importing into LabelSketch itself:

```
labelsketch-dataset-2026-08-14.zip
├── manifest.json
├── images/
│   ├── 3f9a0c12e7b4.jpg
│   └── 8b21d4f0a655.png
└── labels/
    ├── 3f9a0c12e7b4.json
    └── 8b21d4f0a655.json
```

- Every image is renamed to a 12-character content hash (`images/<hash>.<ext>`) — deterministic from the image's own bytes, so the same image always gets the same id, and it carries no trace of your machine's original filename, camera name, or folder structure. If two different images ever hash to the same id (astronomically unlikely, but handled), a `-2`, `-3`, … suffix disambiguates them.
- Each image gets a matching `labels/<hash>.json` with that image's `width`/`height`/`tags`/`annotations` (same pixel + normalized coordinate fields as the flat export above) — one file per sample, which is what most custom PyTorch/TF `Dataset` loaders expect.
- `manifest.json` at the root indexes every image/label pair plus the full class list, so a loader can either read `manifest.json` once or just walk `labels/*.json` directly.
- The **"keep original filenames in manifest"** checkbox (on by default) adds an `original_filename` field to each label and to the manifest, for traceability back to your source files. Uncheck it before exporting if you need the dataset fully anonymized — original filenames (which can carry device names, timestamps, or other identifying info) then never appear anywhere in the archive.
- The ZIP itself is built by hand in vanilla JS (a small "store"/no-compression writer, since images are already JPEG/PNG-compressed) — no library, no network call, consistent with the rest of the tool. It's a standard, valid ZIP any unzip tool can open; LabelSketch does not currently read this format back in (it's a one-way export for training pipelines, not a save format — use **Export/Import Labels JSON** if you need to resume editing later).

## Notes and limitations

- Nothing is saved automatically: export before closing the tab or reloading the page.
- Very large images are downscaled internally (max 2048px on the longest side) for drawing performance; exported pixel coordinates are still rescaled back to the original resolution.
- Editing is geometric only (move/resize/reshape/delete-vertex) — there's no undo history yet, so a bad edit needs a manual fix or a redraw.
- No third-party format export (COCO/YOLO) yet; the native JSON carries both pixel and normalized coordinates so it converts easily with a short script.

## Stack

Vanilla JavaScript + HTML5 Canvas. No frameworks, no build step, no network calls.

## License

MIT
