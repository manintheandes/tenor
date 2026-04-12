# Phase 2: Design Stimuli

Create stimulus images for the study. Each condition gets its own set of images rendered at high resolution. The control condition uses the unmodified product image. Treatment conditions overlay design elements (seals, labels, badges, or other visual treatments) onto the same base image.

## Inputs

Before starting, collect these from the user:

1. **Base product images.** One or more photos of the product being tested. These should be clean product shots with a white or simple background. Accept any common image format (PNG, JPG, WEBP).

2. **Treatment description.** What varies between conditions? Get specifics:
   - What is the visual element? (seal, badge, label, banner, icon overlay, text treatment)
   - What does it communicate? (certification, origin, claim, brand attribute)
   - How many treatment variants? (match the conditions defined in Phase 1)
   - Placement preference? (corner, top, bottom, floating, integrated into packaging)
   - Any reference images or existing brand guidelines?

3. **Distractor product images.** Collect images for the distractor products chosen in Phase 1. These are shown as-is with no treatment applied.

If the user does not have base product images ready, help them find appropriate stock photos or photograph their product. The image quality matters because participants will be studying these closely.

## Stimulus Creation Flow

### Step 1: Design the Treatment Elements

Invoke `/frontend-design` to design the treatment elements (seals, labels, badges, overlays). Provide it with:
- The treatment description from the user
- The base product image (so it can match the visual context)
- The number of variants needed
- Any brand colors, fonts, or style constraints

Ask `/frontend-design` to produce an HTML file containing the treatment elements at high resolution. Each variant should be addressable by a URL fragment (e.g., `#control`, `#treatment-a`, `#treatment-b`) so they can be rendered individually.

### Step 2: Iterate with the User

Show the user the rendered designs. Accept feedback in any form:
- Verbal descriptions ("make the text larger", "use a warmer gold")
- Screenshots (the user may paste or attach annotated screenshots)
- Reference images ("make it look more like this")

Iterate until the user approves. Do not proceed to rendering until you have explicit approval of the design direction.

### Step 3: Build the Render Page

Create an HTML file that composites each treatment element onto the base product image. Structure it so each variant is a separate "frame" that can be activated via URL fragment.

Example structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { background: white; }
  .frame {
    position: relative;
    width: 800px;      /* Match your target output width */
    height: 533px;     /* Match your target output height */
    overflow: hidden;
    display: none;
  }
  .frame:target,
  .frame.active { display: block; }
  .frame img.product {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  .seal-overlay {
    position: absolute;
    top: 8%;
    right: 5%;
    /* Use drop-shadow for clean edges on circular seals */
    filter: drop-shadow(0 4px 14px rgba(0,0,0,0.4));
  }
</style>
</head>
<body>
  <!-- Control: product only, no overlay -->
  <div class="frame" id="control">
    <img class="product" src="product.png" alt="Control">
  </div>

  <!-- Treatment A: product + seal variant A -->
  <div class="frame" id="treatment-a">
    <img class="product" src="product.png" alt="Treatment A">
    <div class="seal-overlay">
      <!-- Treatment A design here -->
    </div>
  </div>

  <!-- Treatment B: product + seal variant B -->
  <div class="frame" id="treatment-b">
    <img class="product" src="product.png" alt="Treatment B">
    <div class="seal-overlay">
      <!-- Treatment B design here -->
    </div>
  </div>
</body>
</html>
```

Use absolute file paths for images in `src` attributes so headless Chrome can resolve them.

### Step 4: Render via Headless Chrome

Render each variant to a hi-res PNG using headless Chrome. Chrome handles fonts, CSS, SVG, gradients, and emoji rendering correctly, which is why we use it instead of pure Python rendering.

**Mac command:**

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless \
  --disable-gpu \
  --hide-scrollbars \
  --screenshot="output.png" \
  --window-size=WIDTH,HEIGHT \
  --force-device-scale-factor=2 \
  --virtual-time-budget=3000 \
  "file://path/to/render.html#variant"
```

**Linux:** Replace the Chrome path with `google-chrome` or `chromium` depending on what is installed. The flags are identical.

Parameter notes:
- `--window-size=WIDTH,HEIGHT` should match the `.frame` dimensions in your CSS (e.g., `800,533` for an 800x533 layout). The output PNG will be 2x these pixel dimensions (1600x1066) because of the scale factor.
- `--force-device-scale-factor=2` produces retina-quality output. This is required. Survey images must be crisp on all devices.
- `--virtual-time-budget=3000` gives the page 3 seconds of virtual time to load fonts, images, and run any JavaScript before capture. Increase if you see missing fonts or blank areas.
- `--screenshot="output.png"` writes the captured image to the specified path. Use descriptive filenames like `treatment-a-full.png`.

Render each variant separately by changing the fragment in the URL:

```bash
# Control
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --screenshot="control-full.png" --window-size=800,533 --force-device-scale-factor=2 --virtual-time-budget=3000 "file:///absolute/path/to/render.html#control"

# Treatment A
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --screenshot="treatment-a-full.png" --window-size=800,533 --force-device-scale-factor=2 --virtual-time-budget=3000 "file:///absolute/path/to/render.html#treatment-a"

# Treatment B
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --screenshot="treatment-b-full.png" --window-size=800,533 --force-device-scale-factor=2 --virtual-time-budget=3000 "file:///absolute/path/to/render.html#treatment-b"
```

Also render zoomed detail views. Create a second HTML layout (or additional frames) that crops to just the treatment element area at a larger scale. These zoomed views are used for seal-specific or label-specific survey questions.

## Image Processing with Pillow

When the user provides images that need cleanup before compositing (e.g., product photos with white backgrounds, seal artwork exported from design tools), use Pillow for processing.

### Background Removal (White Removal)

Remove white or near-white backgrounds pixel by pixel. This is useful for seal artwork or product cutouts:

```python
from PIL import Image

def remove_white_background(input_path, output_path, threshold=240):
    """Remove white/near-white pixels, replacing them with transparency."""
    img = Image.open(input_path).convert("RGBA")
    pixels = img.load()
    w, h = img.size

    for y in range(h):
        for x in range(w):
            r, g, b, a = pixels[x, y]
            if r > threshold and g > threshold and b > threshold:
                pixels[x, y] = (r, g, b, 0)  # Make transparent

    img.save(output_path)
```

Adjust `threshold` based on the image. 240 works for clean white backgrounds. Lower it (e.g., 220) for off-white or slightly textured backgrounds. Inspect the result and adjust.

### Crop to Bounding Box

After removing the background, crop to the content bounding box to eliminate empty space:

```python
def crop_to_content(input_path, output_path):
    """Crop image to the bounding box of non-transparent pixels."""
    img = Image.open(input_path).convert("RGBA")
    bbox = img.getbbox()
    if bbox:
        img = img.crop(bbox)
    img.save(output_path)
```

### Resize

Resize images to a target dimension while preserving aspect ratio. Use `LANCZOS` resampling for downscaling (sharpest results):

```python
def resize_to_width(input_path, output_path, target_width):
    """Resize image to a target width, preserving aspect ratio."""
    img = Image.open(input_path)
    ratio = target_width / img.width
    target_height = int(img.height * ratio)
    img = img.resize((target_width, target_height), Image.LANCZOS)
    img.save(output_path)
```

### Typical Pipeline

For a user-provided seal image that needs compositing:

```python
# 1. Remove white background
remove_white_background("raw-seal.png", "seal-transparent.png", threshold=240)

# 2. Crop to content
crop_to_content("seal-transparent.png", "seal-cropped.png")

# 3. Resize for overlay (e.g., 140px wide for the seal on an 800px product image)
resize_to_width("seal-cropped.png", "seal-final.png", 280)  # 280px = 140px at 2x
```

Then reference `seal-final.png` in the render HTML as an `<img>` inside the overlay `<div>`.

## Showcase Page

After rendering all variants, build an HTML showcase page so the user can review every condition side by side. Invoke `/frontend-design` to create this page.

The showcase page should include:
- All full product views in a grid (one column per condition)
- All zoomed detail views below the full views
- Condition labels above each image (e.g., "Control", "Treatment A: Soft Bee Seal", "Treatment B: Pollinator Protector")
- Clean white background, no distracting styling
- Images at actual rendered resolution (scrollable if needed)

Open the showcase page in a browser for the user to review. Accept feedback and re-render if needed.

## Output Checklist

Before advancing to Phase 3, confirm all of the following:

- [ ] One hi-res PNG per condition showing the **full product view** (product image with treatment overlay, or plain product for control). All at 2x resolution.
- [ ] One hi-res PNG per condition showing the **zoomed detail view** of the treatment element (close-up of the seal, label, or badge). Not needed for control. All at 2x resolution.
- [ ] Distractor product images collected and ready (no treatment applied, used as-is).
- [ ] Showcase page built and reviewed by the user.
- [ ] User has explicitly approved all stimulus variants.

Save all final images to an `images/` directory in the project root. Use clear filenames:
- `control-full.png`
- `treatment-a-full.png`, `treatment-a-detail.png`
- `treatment-b-full.png`, `treatment-b-detail.png`
- `distractor-1.png`, `distractor-2.png`

Do not advance to Phase 3 until the user has approved every image.
