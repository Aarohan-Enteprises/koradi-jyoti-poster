# Jyoti Poster Generator

## Project Overview
A single-page web app for **श्री महालक्ष्मी जगदंबा संस्थान, कोराडी** that generates personalized posters for the **भव्य चैत्र नवरात्र महोत्सव २०२६** (19 March - 28 March). Devotees upload their photo, enter their name and jyoti number, and download a ready-to-share 1080x1080 poster.

## Files
- `index.html` — Complete app (HTML + CSS + JS, no dependencies)
- `template.png` — Original 1080x1080 poster template (source: `C:\Users\murar\Downloads\1080 X 1080.png`)

## Template Layout & Detected Coordinates

All coordinates are in the 1080x1080 pixel space of the template image. These were detected programmatically using Python (PIL + numpy) pixel analysis.

### Photo Area (White Box)
- **Position**: x=522, y=416 to x=1016, y=664
- **Size**: 494 x 248 pixels
- **Detection method**: Scanned for pixels with R>240, G>240, B>240. Found consistent white band from y=418 to y=603 at x=522-1016 (strict threshold), then extended to y=664 using relaxed threshold (R>220, G>220, B>220) which showed white continuing down to the yellow bar.
- The white box is on the **right side** of the template. The goddess/deity image occupies the left side.

### Jyoti Number Circle (ज्योत नं)
- **Center**: (554, 696)
- **Radius**: 91px (96px used for overlay to include white outer border)
- **Detection method**:
  1. Initial attempts using flood-fill on yellow pixels failed due to merging with the yellow bar below.
  2. Detected the **white outer border** of the circle by scanning for white pixels in the range y=600-760, x=380-620.
  3. Found clean **top arc** points by scanning downward at each x position for the first white/content pixel.
  4. Used **three-point circle fit** with symmetric points from the top arc: (495,627), (555,605), (615,628).
  5. Solved analytically: two equations from point pairs → center=(554, 696), radius=91.
  6. Verified: all top arc points are within 1-2px of the fitted circle.
- **Structure** (from center outward): maroon fill with text → yellow border → white outer border
- The circle **overlaps** the bottom-left corner of the photo box. Solution: draw photo first, then redraw the circle region from the template on top.

### Yellow Area (inside circle, for jyoti number text)
- **Position**: x=474-640, y=661-731
- **Center**: (557, 696)
- **Size**: 166 x 70 pixels
- **Detection method**: Scanned for yellow pixels (R>200, G>150, B<60) inside the circle boundary. The yellow area is the lower half of the circle, below the maroon "ज्योत नं :" text.

### Red/Maroon Strip (for devotee name)
- **Position**: x=645-942, y=665-725 (right of circle, below photo)
- **Center used for name**: (830, 695)
- **Detection method**: Scanned for pixels with R>100, G<50, B<50 in the region y=650-850, x=640-1020. Found dense red rows from y=665-720. The strip is a curved banner shape, wider in the middle (~300px at y=710) and narrowing at edges.

## Rendering Pipeline (in order)
1. Draw the full template image
2. Clip to the white box area and draw the devotee photo (with pan/zoom transforms)
3. Redraw the jyoti circle region from the template (clipped to circle) — this makes the circle appear ON TOP of the photo
4. Draw jyoti number text (dark maroon #8b0000 with white outline) on the yellow area
5. Draw devotee name (white #ffffff) on the red strip

## Image Crop/Pan/Zoom System
- On upload, the image is auto-fitted to **cover** the white box (scale = max of width ratio and height ratio)
- Zoom slider (50-300%) scales relative to the auto-fit baseline
- Mouse drag on the canvas pans the image within the clipped box
- Mouse wheel zooms in/out
- Touch events supported for mobile

## Key Design Decisions
- **No frameworks/dependencies** — pure HTML/CSS/JS for easy local use
- **Canvas-based rendering** — all compositing happens on a 1080x1080 canvas, downloaded as PNG
- **Circle overlay technique** — instead of trying to avoid the circle, we let the photo extend behind it and redraw the circle from the template. This gives a natural layered look.
- **Hidden position inputs** — coordinates are hardcoded as hidden inputs (not adjustable by users) since they're precisely calibrated to the template.
