PageXML Line Visualiser
Overview

This Jupyter notebook provides an interactive tool for visualising textual content extracted from document images. Specifically, it overlays the line-level annotations contained within PageXML files onto their corresponding page images, rendering each text line with one of five configurable geometric representations. The tool is designed primarily for quality assessment of optical character recognition (OCR) outputs and for the critical examination of document layout analysis—activities of particular importance to scholars working with historical or archival materials where transcription accuracy is paramount.

Purpose and Use Cases

The visualiser serves several interconnected purposes. First, it permits rapid inspection of OCR line coordinates against the original page images, allowing the practitioner to identify misalignments, missed regions, or systematic errors in the text recognition pipeline. Second, it enables experimentation with alternative representations of textual space—not merely as bounding boxes (the conventional approach) but as phenotypically sensitive renderings that follow the actual contours of the ink or baseline. Third, it facilitates batch processing of entire document collections, generating a corpus of annotated images suitable for further analysis or for publication alongside transcriptions. The tool is thus equally suited to the palaeographer verifying computational transcription as to the historian seeking to understand how OCR systems encode physical and textual information.

Input Requirements

The visualiser expects two categories of input:

PageXML Files — These are XML documents conforming (broadly) to the PAGE (Page Analysis and Ground Truth Elements) schema, typically produced by OCR systems such as Transkribus, Tesseract, or similar tools. Each PageXML file must declare the filename of its corresponding image in the imageFilename attribute of the Page element. The tool parses the following elements within each TextLine:

Coords — A polygon describing the spatial extent of the line (essential for all rendering modes)
Baseline — An optional sequence of points tracing the baseline of the line, used in baseline-band and ink-contour modes
TextEquiv — The recognised text content, extracted to label regions in the output

The namespace http://schema.primaresearch.org/PAGE/gts/pagecontent/2013-07-15 is recognised; the code also falls back to unnamespaced elements for compatibility with non-conformant exports.

Page Images — Corresponding raster images (PNG, JPEG, or any format PIL can read) must be present on disk, with filenames matching those declared in the PageXML. The tool automatically discovers and pairs images with their XML annotations when supplied with a folder or ZIP archive.

Installation and Setup
Dependencies

The notebook requires Python 3.7 or later, along with the following packages:

numpy (numerical operations and interpolation)
Pillow (PIL; image reading and rendering)
matplotlib (preview display)
ipywidgets (interactive controls)

Standard library modules used: xml.etree.ElementTree, pathlib, colorsys, zipfile, re, math, copy, time.

Installation

Install dependencies via pip:

bash
pip install numpy pillow matplotlib ipywidgets

For Conda environments:

bash
conda install numpy pillow matplotlib ipywidgets

The notebook runs in any Jupyter environment (JupyterLab, classic Notebook, or similar) that supports ipywidgets display.

Execution Workflow
Cell 1–3: Initialisation

Run cells 1 through 3 in sequence. These load all necessary imports, instantiate the global state dictionary, and prepare the parsing and geometry functions. Output will confirm "Imports fine" and "Parser ready."

Cell 4–5: Palette and Geometry

These cells define the colour palette (used to assign distinct colours to text regions) and implement the geometric functions for constructing the five line-rendering modes. No user input is required; simply execute them.

Cell 6: File Selection

This cell prompts for a folder path or ZIP archive containing PageXML files and their corresponding images. Supply a path (absolute or relative) to the directory:

python
data_folder = Path('/path/to/your/pagexml/folder')

Alternatively, to work with a ZIP archive:

python
data_folder = Path('/path/to/archive.zip')

The cell will auto-discover all PageXML files, pair them with their images using the imageFilename attribute, and populate a dropdown menu with available pages.

Cell 7: Visualisation and Control

This cell constructs the interactive control panel. Once executed, you will see:

Page dropdown — Select which page to preview
Preview button and scale slider — Render the current page at a specified pixel width (useful for managing computational load on large images)
Visualisation mode — Choose from five rendering options (described below)
Parameter sliders — Fine-tune opacity, ascender/descender heights, ink sensitivity, smoothing, and other geometric properties
Region colour pickers — Assign individual colours to each text region, or enable "Colour by type" to colour regions automatically by their declared type (e.g., paragraph, heading)

Adjust controls in real-time; the preview updates on demand.

Cell 8: Export

Save the current page at full resolution to visualised/{image_name}_lines.png using the "Save PNG" button, or batch-process the entire folder with "Save every page."

Visualisation Modes

Five distinct rendering modes are available, selectable via the radio button panel:

1. Coords Polygon (Default)

The polygon defined by the Coords points of each line is rendered as a filled region, typically following the upper and lower edges of the text. Where the PageXML exporter has computed polygonal coordinates, these will naturally undulate with ascenders and descenders. This mode requires a non-empty Coords polygon and is the most faithful to the original spatial annotation.

2. Ink Contour

For cases where Coords are simple quadrilaterals (unable to follow the actual ink profile), this mode reconstructs the text outline by sampling the page image. For each column across the line, it identifies the highest and lowest dark pixels within a band centred on the baseline, applies median smoothing to eliminate spurious ascenders, and then pads the profile outward. The sensitivity parameter controls the darkness threshold; smoothing and padding parameters are adjustable. This mode is computationally more expensive but yields visually compelling results where ground-truth coordinate polygons are unavailable.

3. Baseline Band

An even-height band offset above and below the baseline (the Baseline points). The band undulates with the baseline itself but maintains a constant width, determined by the ascender and descender parameters. This mode is efficient and useful for quick visual inspection of baseline quality.

4. Straight

A simple rectangle per line, with uniform height and no curvature. Fastest to render; useful for high-level overview or for testing parameter changes.

5. Letters

The strokes of the text themselves, extracted via an ink stencil derived from the page image. This mode is visually striking but expensive to compute and most useful at high resolution. Requires both Coords and the page image.

Key Parameters Explained
Opacity — Transparency of the rendered line overlay (0–1)
Pearl effect — Applies a slight luminance pattern for aesthetic effect
Ascender / Descender — Height in pixels above and below the baseline (baseline band mode)
Sensitivity — Ink darkness threshold for ink-contour mode (higher = darker pixels required)
Padding — Pixels added around detected ink to prevent tight clipping
Smoothing — Median-filter width for smoothing the ink profile (ink-contour mode)
Crowd mitigation — Erosion applied to prevent ink from overlapping across adjacent lines
Max ascender / descender — Upper limits (pixels) on ascender and descender heights, preventing extreme outliers
Thicken — Additional stroke width applied uniformly to all lines
Coords outline — If enabled, draws the bounding polygon with a thin border
Outline width — Thickness (pixels) of that border
Colour by type — Automatically assigns colours based on region type rather than manual assignment
Technical Notes
Coordinate Scaling

The tool automatically detects if a page has been resampled since transcription (by comparing declared page dimensions in the PageXML against the actual image size) and scales all coordinates proportionally. This ensures that annotations remain aligned even after image resizing.

Caching and Performance

Preview rendering occurs at the width specified by the scale slider; full-resolution rendering happens only on export. Previews are cached to avoid unnecessary recomputation during rapid parameter adjustments. For high-resolution images (3000+ pixels wide), preview-width rendering takes fractions of a second, whereas full-resolution can require several seconds.

Namespace Handling

The code gracefully handles both namespaced (PAGE schema) and non-namespaced XML variants. The _find helper function searches first for namespaced elements before falling back to unnamespaced equivalents.

Zip Archive Support

When supplied with a ZIP archive, the tool extracts PageXML files and discovers corresponding images within the archive, or looks for images in the extraction directory. This is useful for working with Transkribus or similar export packages.

Examples
Basic Usage
python
# Cell 6: Point to a folder
data_folder = Path('./my_documents')

Then run cell 7 to display the control panel. Select a page, choose "Coords polygon" mode, and click Preview.

Quality Assessment Workflow
Select "Ink contour" mode for a realistic rendering of text boundaries
Adjust sensitivity and padding until line contours closely follow the actual letter strokes
Preview several pages to ensure consistency
Click "Save every page" to generate a full set of annotated images
Import the resulting PNG files into a document management system or annotation tool for manual review
Batch Production

To visualise and export all pages in a collection with identical settings:

Load your data folder (cell 6)
Configure all parameters (cell 7)
Click "Save every page"
Outputs appear in a visualised/ subdirectory
Limitations and Caveats
The tool assumes that the imageFilename attribute in each PageXML is accurate and points to an image file in the same directory or archive
Performance on images larger than 4000 pixels in width or height may be slow, particularly in "Letters" mode
The ink-contour mode depends on image contrast; low-contrast or degraded scans may produce uninformative results
Baseline points must be ordered left-to-right for correct interpolation; erratic or reversed baselines may cause rendering artifacts
The tool does not currently handle multi-page PDFs; each page must be supplied as a separate image file
Output

All exported images are saved to a visualised/ directory in the current working directory. Filenames follow the pattern {original_image_name}_lines.png. Images are saved with PNG compression level 3 (moderate compression for file size without excessive encoding overhead).

Citation and Attribution

If you employ this tool in published research, please note its use in your methods section. A suggested formulation might read: "Text line boundaries were visualised using a custom PageXML overlay tool, permitting manual verification of OCR coordinate accuracy."

Further Development

Potential enhancements for future versions include:

Support for word-level and character-level annotations
Export to vector formats (SVG) for further editing
GPU acceleration for ink-contour rendering
Direct integration with Transkribus API for live preview of OCR outputs
Configurable annotation layers (metadata, confidence scores, etc.)
Troubleshooting

No pages appear in the dropdown: Check that the folder contains both PageXML files and corresponding images, and that the imageFilename attribute in each PageXML is accurate.

Preview is very slow: Reduce the preview width using the scale slider, or switch to "Straight" mode (faster rendering).

Lines are misaligned with text: Verify that the page has not been resampled since transcription. Check the declared dimensions in the PageXML against the actual image size. If they differ, the tool will attempt automatic scaling, but extreme differences may indicate a data integrity issue.

Ink-contour mode produces blank output: The image may be too low-contrast. Increase the sensitivity parameter to capture fainter ink, or adjust the padding and smoothing parameters.
