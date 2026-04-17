# Presentation Animation Quality Checklist

Guidelines for creating manim (or similar) animated figures for academic presentations.

## First Question: Presentation Slide or Standalone Video?

**Always clarify this before starting.** The answer changes everything about spacing.

- **Presentation slide embed**: The animation will be placed inside a PowerPoint slide alongside titles, body text, and design insight boxes. It occupies maybe 50-70% of the slide area. **Compactness is critical.** Every pixel of white margin shrinks the actual content. Use minimal padding, tight axis labels, no title (the slide has one), and compact legends.
- **Standalone video** (e.g., YouTube, video figure): Full-frame is fine. Titles, generous spacing, and breathing room are appropriate.

**Default to presentation-compact unless told otherwise.**

## Before Rendering: Layout Verification

### Overlap Prevention
- **Legend vs axes**: Calculate the exact pixel/unit position of the legend box and verify it doesn't overlap with any axis line, axis labels, or tick marks. Legend should have >= 0.5 units clearance from axes.
- **Legend vs data**: Check that the legend doesn't cover any data points, especially at the first or last session where points cluster.
- **Endpoint labels vs each other**: When multiple lines converge (e.g., two WPM values within 2 units), stagger labels vertically or use different directional offsets (UP, DOWN, RIGHT). **CRITICAL: Always sort data values and compute pairwise gaps. If any two values are within 1.5 data units, they WILL overlap when annotated. Push the lower one DOWN and the upper one UP by at least 0.25 manim units each.** Common trap: values like 16.60 and 16.64 look different in the data array but render at virtually the same pixel — always check numerically, don't eyeball the data.
- **Axis tick labels vs annotations**: If placing colored value annotations near an axis (e.g., WPM values on the y-axis), either hide the default axis tick numbers or ensure annotations are offset enough to not overlap with them.
- **Twin y-axis ticks vs axis line**: When using a twin/secondary y-axis, shift the tick number labels further outward (add RIGHT shift) so they don't sit on top of the axis line itself.
- **Box plot labels vs each other**: Use abbreviated condition names (e.g., SP, SP-B) when horizontal space is tight. Test that names don't overlap at the font size chosen.

### Compactness (for presentation embeds)
- **No title in the animation** — the slide already has one. Titles eat vertical space.
- **Legend**: Use inline technique names on Session 1 dots → pause → fade to compact legend. Place legend INSIDE the plot area, not above or beside it. If using a separate legend box, it must sit in empty data space (e.g., upper-left where values are low).
- **Axis labels**: Use `buff=0.1-0.15` not `buff=0.3-0.4`. Every 0.1 unit of buff is wasted space at presentation scale.
- **Panel titles** (for multi-panel): Place directly above axes with `buff=0.08-0.12`, not 0.25+.
- **Panel gaps**: For side-by-side panels, use `gap=0.4-0.6`, not 1.0+.
- **Outer margins**: The animation should fill the frame edge-to-edge with minimal breathing room. Target content spanning >= 85% of frame width and >= 80% of frame height.
- **Axes dimensions**: Make axes as large as possible — `x_length` and `y_length` should fill most of the available space.

### Text Readability
- **Minimum font sizes** for 1080p presentation video:
  - Title: >= 36pt
  - Axis labels: >= 24pt
  - Data labels / annotations: >= 20pt
  - Legend items: >= 20pt
  - Axis tick numbers: >= 20pt
  - Key result callouts (e.g., "37% shorter"): >= 28pt, bold
- **Contrast**: On white backgrounds, use dark colors (BLACK, GREY_D). On dark backgrounds, use WHITE or bright colors. Never use light gray text on white.
- **Color distinctness**: Use a colorblind-friendly palette (e.g., Dark2 from ColorBrewer). Test that all conditions are visually distinguishable. Avoid pure blue (#0000FF) — it's hard to see on projectors.

### Coordinate Systems
- **Check axis orientation**: Verify that the data's coordinate system matches manim's. Common trap: data with y-inverted (screen coordinates where y=0 is top) needs careful handling. Test with a known point (e.g., "QWER row should be at the top of the keyboard").
- **Traces/paths must use the same transform as the background**: If you flip y for the keyboard, you must also flip y for the traces drawn on it.

## During Animation: Flow & Timing

### Pacing
- **Introduce elements progressively**: Don't show everything at once. Typical flow: axes → data → annotations → callouts.
- **Session/condition introduction**: When showing multiple conditions for the first time, display technique names ON the data points first, pause 1-2 seconds, then swap to a compact legend. This teaches the audience the color mapping.
- **Pause after key reveals**: After the main result appears (e.g., the final session values), hold for >= 1 second before any transition.

### Transitions
- **Session label changes**: Make session transitions obvious — use bold text, larger font, and a visual cue (flash, slide-up animation). Gray text for session numbers is too subtle.
- **Zoom transitions**: When zooming into a subset of data (e.g., zooming into Session 5), first zoom smoothly (1-1.5s), then widen the camera to fit the incoming content (box plot, etc.) BEFORE drawing that content.
- **Cleanup stale elements**: When transitioning between sessions or views, explicitly FadeOut all elements from the previous state. Never leave orphaned labels, lines, or dots from a prior session.

### Z-ordering
- **Dots above lines**: Set z_index=10 for data point dots and z_index=1 for connecting lines. Draw lines first, then dots, so dots always render on top.
- **Annotations above data**: Callout boxes, arrows, and text annotations should have high z_index.

## After Rendering: Verification

### Camera Bounds Check
- **Mathematically verify** that all content fits within the camera frame, especially after zoom transitions. For 16:9 aspect ratio: `frame_height = frame_width * 9/16`. Check:
  - Top edge: highest element (title, whisker label) < frame_center_y + frame_height/2
  - Bottom edge: lowest element (condition labels) > frame_center_y - frame_height/2
  - Left/right edges: axis labels and secondary axes fit within frame_center_x ± frame_width/2
- **If anything is clipped**: Increase camera width, reduce content dimensions, or reposition content toward center. Then re-verify.

### Visual Sense Check
- **Pareto frontier lines**: Use staircase (horizontal-then-vertical straight line segments), not smooth curves, when there are only 2-3 points. Curves imply interpolation that doesn't exist.
- **Data consistency**: Verify that the numbers shown in annotations match the data arrays in the script. Off-by-one-decimal errors are common.
- **Start/end dots**: If showing trace paths, green=start and red=end is the convention. Make sure they're visible (z_index=10, not covered by the path).

## Common Patterns

### Line Chart → Box Plot Transition
1. Draw line chart progressively across sessions
2. Show endpoint labels at final session
3. Pause for audience
4. Zoom camera into final session region
5. Fade out all line chart elements except final session dots
6. Widen camera to fit box plot dimensions (verify bounds!)
7. Show box plot axes
8. Animate final-session dots sliding to box plot positions
9. Draw box plot elements (boxes, whiskers, medians, caps, labels)

### Overlay Comparison (e.g., two trace paths)
1. Draw the baseline/reference condition first in its color
2. Pause briefly
3. Fade baseline to gray + low opacity
4. Draw the comparison condition on top in its color
5. Show the metric (e.g., "37% shorter") prominently

### Champion Highlighting
- Use a colored ring (Circle) around the champion condition's dot
- Ring color should match the champion's condition color (not yellow on white backgrounds)
- Ring should appear early (with technique name labels) so the audience tracks it
