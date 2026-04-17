---
name: presentations
description: Build publication-quality academic/HCI presentation slides programmatically using python-pptx and Manim. Covers slide structure, contrast-safe color choices for any background, animated data figures, presenter notes, and HCI-specific storytelling conventions (Result + Insight pairs, punchlines). Use when the user asks to create, modify, or animate slides for a conference video or talk.
---

# Academic Presentation Builder

Creates polished conference presentation slides (PowerPoint) with embedded Manim animations, proper contrast for any background color, structured presenter notes, and HCI-specific storytelling patterns.

---

## Quick Start

1. **Read the paper** — load the abstract, results, and discussion into context before touching slides
2. **Read the existing pptx** — understand current slide count, background color, font sizes, color palette
3. **Check contrast** — determine background color first; every text/element choice follows from it
4. **Write Manim scripts** — one scene per animated result, background matches slides
5. **Render animations** — MP4 for embedded video (click-to-play), PNG for static
6. **Build slides with python-pptx** — programmatic construction for consistency
7. **Write presenter notes for EVERY slide** — narration is mandatory, not optional; read ALL existing notes before writing new ones to ensure continuity (see §6)

---

## 1. Contrast — Always Check First

**Rule:** Determine the slide background color before choosing any other color. Every subsequent choice must pass WCAG contrast tests.

### Contrast Ratio Minimums
| Use | Minimum ratio | Recommended |
|-----|--------------|-------------|
| Body text (normal) | 4.5 : 1 | 7 : 1 |
| Large text (≥18pt bold or ≥24pt) | 3 : 1 | 4.5 : 1 |
| UI components / data marks | 3 : 1 | 4.5 : 1 |
| Decorative only | none | — |

Calculate: `contrast = (L1 + 0.05) / (L2 + 0.05)` where L is relative luminance.  
Quick tool: https://webaim.org/resources/contrastchecker/

### Safe Palettes by Background

**Dark/Black background (#0D0D0D or #000000)**
```
Titles:            #FFFFFF  (white)          ratio ~21:1  ✓
Section headers:   #90CAF9  (light blue)     ratio ~8:1   ✓
Body/quotes:       #BDBDBD  (light gray)     ratio ~8:1   ✓
Slide numbers:     #757575  (medium gray)    ratio ~4:1   ✓ (large)
Accent blue:       #42A5F5  (mid blue)       ratio ~5:1   ✓
Accent green:      #66BB6A  (mid green)      ratio ~5:1   ✓
Accent red:        #EF5350  (mid red)        ratio ~4:1   ✓ (large only)
AVOID:             #1565C0  (dark blue)      ratio ~1.7:1 ✗
AVOID:             #212121  (near-black)     ratio ~1.2:1 ✗
AVOID:             #2E7D32  (dark green)     ratio ~2.1:1 ✗
```

**Light/White background (#FFFFFF or #F5F5F5)**
```
Titles:            #212121  (near-black)     ratio ~16:1  ✓
Body text:         #424242  (dark gray)      ratio ~9:1   ✓
Accent blue:       #1565C0  (dark blue)      ratio ~6:1   ✓
Accent green:      #2E7D32  (dark green)     ratio ~5:1   ✓
AVOID:             #90CAF9  (light blue)     ratio ~2:1   ✗
AVOID:             #BDBDBD  (light gray)     ratio ~2:1   ✗
```

**Cream/finding strip (#FFF3E0)**
```
"Finding" label:   #0D47A1  (deep blue)     ratio ~9:1   ✓
"Insight" label:   #1B5E20  (deep green)    ratio ~9:1   ✓
Body text:         #212121  (near-black)     ratio ~16:1  ✓
```

### Colorblind Safety
- Never use red/green as the only differentiator — add shape, pattern, or label
- Safe pairs: blue + orange, blue + yellow, purple + yellow
- Test with: https://www.color-blindness.com/coblis-color-blindness-simulator/

---

## 2. Slide Structure & Layout

### Standard Dimensions
- **16:9 widescreen:** 13.33" × 7.50" (most conferences)
- **4:3 standard:** 10" × 7.5" (older venues)

### Consistent Layout Elements (python-pptx positions)
```python
# Title bar — top left, always same position
title:    pos=(0.5", 0.25"), size=(12.33" × 0.85"), font=36pt bold

# Finding/Insight strip — bottom, always same
strip:    pos=(0.5", 6.0"),  size=(12.33" × 1.10")

# Slide number — bottom right
number:   pos=(12.5", 7.0"), size=(0.70" × 0.40"), font=16pt

# Content area (between title and strip)
content:  top=1.1", bottom=6.0"  →  available height ≈ 4.9"
```

### Typography Rules
| Element | Size | Weight | Notes |
|---------|------|--------|-------|
| Section header | 48–56pt | Bold | Centered, use accent color |
| Slide title | 34–40pt | Bold | Top-left anchor |
| Body text | **24pt minimum** | Regular | Hard floor — never below 24pt |
| Quotes | 26–30pt | Italic | Different color from body |
| Attribution | 24pt | Regular | Subdued color — still 24pt minimum |
| Captions | 24pt | Regular | Below figures |
| Strip label | 24–26pt | Bold | "Finding" / "Insight" |
| Strip body | 24pt | Regular | One sentence max |

**Hard rule: 24pt is the absolute minimum for any text in a slide.** Anything smaller is unreadable when projected. Slide numbers and decorative labels are the only exception.

**Font choices:** Use sans-serif throughout (Calibri, Helvetica, Inter, Source Sans Pro).  
Avoid: Times New Roman, decorative fonts, more than 2 typefaces per deck.

---

## 3. HCI Presentation Storytelling

### The Result + Insight Pattern
Every result slide pairs with an insight slide. This is the core unit:

```
[Result slide]
  Title: "Study X: Condition Y"
  Content: Animated data visualization
  Strip: "Finding — [one-sentence quantitative result with p-value]"

[Insight slide]
  Title: "Study X: Condition Y"  ← same title keeps viewer oriented
  Content: Resident/participant quote (centered, large, italic)
  Strip: "Insight — [design/behavioral implication in plain language]"
```

### Narrative Arc for 5-minute Video
```
0:00 – 0:30   Title + motivation (problem in one sentence)
0:30 – 1:30   Background + method (pipeline, study design)
1:30 – 2:00   Study 2 results (2 Result+Insight pairs × ~15s each)
2:00 – 2:10   Section transition to Study 3
2:10 – 3:30   Study 3 results (3 Result+Insight pairs × ~25s each)
3:30 – 4:00   Design implications
4:00 – 4:30   Conclusion + future work
4:30 – 5:00   Questions / acknowledgments
```

### Punchlines
Bold, standalone takeaway sentences. Place as the first sentence of a result paragraph in papers; use as the strip content in presentations.
- Keep to one sentence
- State the claim, not the evidence ("AOI encodes lasting habits" not "post-block carryover p<0.001")
- Evidence appears in the "Finding" strip; interpretation appears in "Insight"

### Quotes
Always pair a quote with an attribution (role, not name, for blind papers).
```python
quote_text = '"The text box was incredibly helpful — I could confirm or reject\nentries rather than starting from scratch."'
attribution = "— PGY-3 Resident"
```

---

## 4. Manim Animations

### When to Animate
- **Always animate:** bar charts (bars grow in), scatter plots (dots appear), flow diagrams (arrows draw)
- **Static is fine:** tables, screenshots, interface mockups, Venn diagrams with few elements
- **Never animate:** text-heavy slides (distracts), complex multi-panel figures (too busy)

### Setup
```bash
pip install manim
manim --version   # should show ManimCE vX.X
```

### Script Template
```python
from manim import *

class MyScene(Scene):
    def construct(self):
        # 1. Set background to MATCH the slide background
        self.camera.background_color = ManimColor("#0D0D0D")  # dark slides
        # self.camera.background_color = WHITE                 # light slides

        # 2. All text must pass contrast against this background
        # 3. Build your scene...
        pass
```

### Rendering Commands
```bash
# Static final frame (PNG) — for embedding as image
manim -s --format=png -r 1920,1080 scene.py ClassName
# Output: media/images/scene/ClassName_ManimCE_*.png

# Full animation (MP4) — for embedding as video in pptx
manim --format=mp4 -qm scene.py ClassName
# Output: media/videos/scene/720p30/ClassName.mp4

# Quality flags: -ql (480p), -qm (720p), -qh (1080p), -qk (4K)
```

### Bar Chart Pattern (dark background)
```python
from manim import *

class BarChart(Scene):
    def construct(self):
        self.camera.background_color = ManimColor("#0D0D0D")

        labels = ["Condition A", "Condition B", "Condition C"]
        values = [2.5, 3.1, 4.8]
        colors = [ManimColor("#90A4AE"), ManimColor("#42A5F5"), ManimColor("#66BB6A")]

        axes = Axes(
            x_range=[0, len(labels)+1, 1],
            y_range=[0, max(values)*1.3, 1],
            x_length=8, y_length=5,
            axis_config={"color": WHITE, "include_tip": False},
            y_axis_config={"include_numbers": True, "font_size": 26, "color": WHITE},
        )
        y_label = Text("Your metric", font_size=28, color=WHITE).rotate(PI/2).next_to(axes, LEFT, buff=0.4)
        self.add(axes, y_label)

        for i, (label, val, color) in enumerate(zip(labels, values, colors)):
            x_pos = i + 0.8
            bar_h = axes.y_axis.n2p(val)[1] - axes.y_axis.n2p(0)[1]
            bar = Rectangle(
                width=0.8, height=bar_h,
                fill_color=color, fill_opacity=0.92,
                stroke_color=ManimColor("#0D0D0D"), stroke_width=1
            ).move_to(axes.c2p(x_pos, 0), aligned_edge=DOWN)
            bar.shift(UP * (axes.y_axis.n2p(0)[1] - bar.get_bottom()[1]))
            vl = Text(f"{val}", font_size=26, color=WHITE, weight=BOLD).next_to(bar, UP, buff=0.12)
            xl = Text(label, font_size=22, color=ManimColor("#BDBDBD")).move_to(axes.c2p(x_pos, 0) + DOWN*0.5)

            self.play(GrowFromEdge(bar, DOWN), run_time=0.5)
            self.play(Write(vl), Write(xl), run_time=0.3)

        self.wait(1.0)
```

### "Before → After" Number Pattern
```python
class BeforeAfter(Scene):
    def construct(self):
        self.camera.background_color = ManimColor("#0D0D0D")

        before = Text("7.2 s", font_size=110, color=ManimColor("#BDBDBD"), weight=BOLD).shift(LEFT*3.5)
        b_lbl  = Text("Before", font_size=28, color=ManimColor("#9E9E9E")).next_to(before, DOWN)

        arrow = Arrow(LEFT*0.7, RIGHT*0.7, color=ManimColor("#64B5F6"), stroke_width=7)
        pct   = Text("−67%\np = 0.023", font_size=36, color=ManimColor("#64B5F6"), weight=BOLD).next_to(arrow, UP)

        after = Text("2.4 s", font_size=110, color=ManimColor("#64B5F6"), weight=BOLD).shift(RIGHT*3.5)
        a_lbl = Text("After", font_size=28, color=ManimColor("#9E9E9E")).next_to(after, DOWN)

        self.play(Write(before), FadeIn(b_lbl), run_time=0.5)
        self.play(Create(arrow), Write(pct), run_time=0.5)
        self.play(Write(after), FadeIn(a_lbl), run_time=0.5)
        self.wait(1.0)
```

### Annotation Patterns
```python
# Brace showing difference between two bars
brace = BraceBetweenPoints(
    axes.c2p(x_ref, val_low),
    axes.c2p(x_ref, val_high),
    direction=RIGHT, color=ManimColor("#90CAF9")
)
label = Text("+59% vs best alone", font_size=26, color=ManimColor("#90CAF9"), weight=BOLD)
label.next_to(brace, RIGHT, buff=0.2)
self.play(Create(brace), Write(label), run_time=0.8)

# Arrow callout
arr = Arrow(start_point, end_point, color=ManimColor("#A5D6A7"), stroke_width=4)
ann = Text("Key finding (p<0.001)", font_size=28, color=ManimColor("#A5D6A7"), weight=BOLD)
ann.next_to(arr.get_start(), UP)
self.play(Create(arr), Write(ann), run_time=0.7)
```

---

## 5. python-pptx: Building Slides

### Setup
```bash
pip install python-pptx pillow
```

### Core Helper Functions
```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN

def i(x): return Inches(x)

def set_bg(slide, hex_color="#0D0D0D"):
    r, g, b = int(hex_color[1:3],16), int(hex_color[3:5],16), int(hex_color[5:7],16)
    fill = slide.background.fill
    fill.solid()
    fill.fore_color.rgb = RGBColor(r, g, b)

def add_title(slide, text, font_size=36, color="#FFFFFF"):
    r,g,b = int(color[1:3],16), int(color[3:5],16), int(color[5:7],16)
    tb = slide.shapes.add_textbox(i(0.5), i(0.25), i(12.33), i(0.85))
    p = tb.text_frame.paragraphs[0]
    run = p.add_run(); run.text = text
    run.font.size = Pt(font_size); run.font.bold = True
    run.font.color.rgb = RGBColor(r,g,b)

def add_finding_strip(slide, label_text, body_text, kind="Finding"):
    # kind = "Finding" (blue label) or "Insight" (green label)
    rect = slide.shapes.add_shape(1, i(0.5), i(6.02), i(12.33), i(1.10))
    rect.fill.solid(); rect.fill.fore_color.rgb = RGBColor(0xFF,0xF3,0xE0)
    rect.line.color.rgb = RGBColor(0xD0,0xC0,0xA0); rect.line.width = Pt(0.5)
    tf = rect.text_frame; tf.word_wrap = True
    p1 = tf.paragraphs[0]
    r1 = p1.add_run(); r1.text = label_text; r1.font.size = Pt(24); r1.font.bold = True
    r1.font.color.rgb = RGBColor(0x0D,0x47,0xA1) if kind=="Finding" else RGBColor(0x1B,0x5E,0x20)
    p2 = tf.add_paragraph()
    r2 = p2.add_run(); r2.text = body_text; r2.font.size = Pt(20)
    r2.font.color.rgb = RGBColor(0x21,0x21,0x21)

def add_video(slide, mp4_path, poster_path, left=0.45, top=1.15, width=12.43, height=4.75):
    slide.shapes.add_movie(mp4_path, i(left), i(top), i(width), i(height),
                           poster_frame_image=poster_path, mime_type="video/mp4")

def add_presenter_notes(slide, text):
    slide.notes_slide.notes_text_frame.text = text

def delete_slide(prs, idx):
    """Properly remove a slide by index."""
    from pptx.oxml.ns import qn
    sldIdLst = prs.slides._sldIdLst
    elem = sldIdLst[idx]
    r_id = elem.get(qn('r:id'))
    sldIdLst.remove(elem)
    prs.slides.part.drop_rel(r_id)
```

### Section Header Slide
```python
def section_slide(prs, title, subtitle, slide_num, bg="#0D0D0D", title_color="#90CAF9"):
    slide = prs.slides.add_slide(prs.slide_layouts[6])  # blank
    set_bg(slide, bg)
    tb = slide.shapes.add_textbox(i(1.0), i(2.2), i(11.33), i(1.5))
    tf = tb.text_frame; tf.word_wrap = True
    p = tf.paragraphs[0]; p.alignment = PP_ALIGN.CENTER
    r = p.add_run(); r.text = title
    r.font.size = Pt(52); r.font.bold = True
    r,g,b = int(title_color[1:3],16),int(title_color[3:5],16),int(title_color[5:7],16)
    r.font.color.rgb = RGBColor(r,g,b)
    if subtitle:
        tb2 = slide.shapes.add_textbox(i(1.0), i(3.9), i(11.33), i(0.8))
        p2 = tb2.text_frame.paragraphs[0]; p2.alignment = PP_ALIGN.CENTER
        r2 = p2.add_run(); r2.text = subtitle; r2.font.size = Pt(26)
        r2.font.color.rgb = RGBColor(0xBD,0xBD,0xBD)
    return slide
```

### Embedding Video (MP4)
```python
from PIL import Image

def make_black_poster(output_path, width=1280, height=720):
    img = Image.new("RGB", (width, height), (13, 13, 13))
    img.save(output_path)

# Usage
make_black_poster("poster.png")
add_video(slide, "animation.mp4", "poster.png")
```

---

## 6. Presenter Notes — MANDATORY on Every Slide

**Hard rule: Every slide MUST have presenter notes.** No exceptions — including title slides, section headers, and conclusion slides. Presenter notes ARE the narration script for the video recording. A slide without notes is a slide the presenter will stumble through.

### Before Writing Any Notes

**Read the full paper first.** The narration must be grounded in the actual paper's claims, numbers, and framing — not improvised or paraphrased from slide content alone.

**Read ALL existing slide notes before writing new ones.** The narration must flow as a continuous story. Each slide's notes must:
1. Pick up naturally from where the previous slide's notes ended
2. Transition smoothly into what the next slide will show
3. Never repeat a sentence or finding already stated in a prior slide's notes
4. Maintain consistent terminology (if slide 3 says "Correct Diagnoses per Minute," slide 8 must not switch to "diagnostic throughput" without explanation)

### Context Loading Checklist (before writing notes)

Before writing presenter notes for any slide, you MUST have in context:
- [ ] The paper's abstract (for framing and key claims)
- [ ] The paper's results sections (for exact numbers, p-values, effect sizes)
- [ ] The paper's discussion punchlines (for insight framing)
- [ ] ALL existing presenter notes from earlier slides (for continuity)
- [ ] The slide's own visual content (title, strip text, animation description)
- [ ] The NEXT slide's content (to write the transition sentence)

### Note Structure by Slide Type

**Title / Opening slide:**
```
[Greeting / hook — one sentence stating the problem]
[What this work is — one sentence naming the system]
[What the video will cover — "In this talk, we'll show..."]
```

**Section header slide (8 seconds max):**
```
[Transition from previous section: "Now let's look at..." or "Having established X, we turn to Y..."]
[One sentence previewing what this section covers and why it matters]
```

**Finding slide (with animation, 25–30 seconds):**
```
[Click to start animation]

[Opening: what the viewer is about to see — "Here are the results for..." or "Watch the three bars..."]

[As animation plays]: narrate what's happening as each element appears.
Use present tense: "The bars show...", "Notice that...", "The green bar reaches..."

[Key finding: state the number and p-value conversationally]
"That's a 52 percent carryover — statistically significant at p less than 0.001."

[Bridge to next slide: "Residents told us why..." or "But this raised a question..."]
```

**Insight slide (with quote, 20 seconds):**
```
[Reference the quote on screen — read it or paraphrase]
"As this PGY-3 put it: [abbreviated quote]..."

[Explain why this matters — connect the quote to the design principle]
"This tells us that [insight in plain language]."

[Transition to next result or section]
"But [modality X] doesn't address [other bottleneck]. That's where [modality Y] comes in."
```

**Conclusion slide (25–30 seconds):**
```
[Signal closing: "Let me close with..." or "To summarize..."]

[Three take-aways — one sentence each, mirroring the slide bullets]

[Future work — one sentence]

[Sign-off: "Thank you."]
```

### Writing Rules

- **Write for the ear, not the eye.** Short sentences, active voice, no LaTeX notation.
  - Say "p equals 0.023" not "p=0.023"
  - Say "52 percent" not "52%"
  - Say "two and a half times more" not "2.5×"
- **Include ALL numbers you'd say out loud.** The presenter should never need to glance at the slide for a number.
- **Never start two consecutive slides' notes with the same sentence structure.** Vary openings.
- **End every note with a transition** — the last sentence must set up the next slide.
- **Mark animation cues** with `[brackets]`: `[Click to start animation]`, `[As the bars grow in]`, `[When the arrow appears]`
- **Timing:** One slide ≈ 15–30 seconds for results, 8 seconds for section headers, 25–30 seconds for conclusion.

### Continuity Test

After writing all notes, read them aloud in sequence (without looking at slides). They should sound like a coherent talk, not a series of disconnected captions. Specifically check:
1. Does each transition connect to the next slide's content?
2. Are any findings stated twice?
3. Is the emotional arc clear? (problem → individual solutions → combined solution → why it matters)
4. Would a listener who can't see the slides understand the story?

---

## 7. Data Visualization Principles for Slides

### Bar Charts
- Sort bars by value when there's no natural order
- Label values directly on/above bars (avoid legends when possible)
- Highlight the "hero" bar with a brighter color or annotation
- Use a brace/bracket to show a difference between two bars
- Include p-value and direction of comparison in the annotation

### Tables in Slides
- Maximum 5 columns × 6 rows for readability
- Bold the most important cell in each row
- Use light alternating row shading (not zebra stripes — too distracting)
- Tables in slides should answer ONE question; more → split into multiple slides

### Avoiding Common Mistakes
| Mistake | Fix |
|---------|-----|
| Too much text | Max 6 words per bullet; use notes for full sentences |
| Tiny font | Never below 20pt in a slide meant to be projected |
| Chartjunk | Remove gridlines, legends (label directly), 3D effects |
| Animations for show | Only animate when it adds meaning (data appearing sequentially) |
| Inconsistent colors | Same color = same concept across ALL slides |
| All-caps headers | Use title case; all-caps is harder to read at speed |

---

## 8. Slide Reduction Rules

If a slide feels too busy:
1. **One idea per slide** — split into two slides if needed
2. **Replace text with visuals** — a quote replaces a bullet list
3. **Move to notes** — methodology details belong in presenter notes, not slide body
4. **Use progressive reveal** — animate content in rather than showing all at once
5. **White/negative space** — empty space is not wasted space; it guides the eye

---

## 9. File Management

```
project/
├── video_assets/
│   ├── scene_name.mp4        # Manim animation
│   ├── scene_name_poster.png # Black thumbnail for video placeholder
│   └── scene_name_static.png # Last frame (for fallback)
├── manim_scripts/
│   ├── scene1.py
│   └── scene2.py
└── build_slides.py           # python-pptx construction script
```

**Always save a backup before modifying:**
```python
import shutil
shutil.copy("presentation.pptx", "presentation_backup.pptx")
```

**Render all Manim scenes in one command:**
```bash
for f in manim_scripts/*.py; do
  cls=$(grep "^class " $f | head -1 | awk '{print $2}' | tr -d ':')
  manim --format=mp4 -qm $f $cls
done
```

---

## 10. Text Readability in Manim Animations

Manim uses its own font size scale (not PowerPoint points). The equivalent of PowerPoint 24pt in a 1920×1080 Manim scene is approximately **font_size=26–28** in Manim units.

**Manim font_size equivalents (at 1920×1080, default scene scaling):**
| Use | Manim font_size | PowerPoint equivalent |
|-----|----------------|----------------------|
| Axis tick numbers | 26–28 | ~24pt ✓ |
| Axis labels | 28–30 | ~26pt ✓ |
| Bar value labels | 26–28 | ~24pt ✓ |
| X-axis category labels | 22–24 | ~20pt — borderline |
| Annotation callouts | 28–32 | ~26pt ✓ |
| Large display numbers | 80–120 | decorative |

**Rule:** Any informational text in a Manim scene must use `font_size ≥ 26`. At 720p render (`-qm`), scale down by ~0.75 — use `font_size ≥ 30` for safety.

**Visual inspection required:** After rendering, open the PNG/MP4 and check that all text is clearly legible at normal viewing distance. If any text is hard to read, increase font_size by 4 and re-render.

---

## 11. Boundary / Clipping Check

Before finalising any slide, verify that no element extends outside the slide canvas.

**python-pptx safe zone:**
```python
SLIDE_W = 13.33  # inches
SLIDE_H = 7.50   # inches

# Check all shapes
for shape in slide.shapes:
    right  = (shape.left + shape.width)  / 914400  # EMU → inches
    bottom = (shape.top  + shape.height) / 914400
    if right > SLIDE_W:
        print(f"WARNING: {shape.name} clips right edge by {right - SLIDE_W:.2f}\"")
    if bottom > SLIDE_H:
        print(f"WARNING: {shape.name} clips bottom edge by {bottom - SLIDE_H:.2f}\"")
```

**Manim safe zone:** Leave at least 0.3 units of padding on all edges. Content that reaches the edge of the Manim canvas will be cropped when PowerPoint scales the video object.

**Strip conflict check:** The Finding/Insight strip occupies `top=6.0"` to `top=7.1"`. Any content body placed below `5.9"` will be hidden behind the strip.

---

## 12. Quick Checklist Before Recording

### Visual
- [ ] Every text element passes contrast ratio ≥ 4.5:1 against its background
- [ ] **All slide text ≥ 24pt** (hard minimum — no exceptions except slide numbers)
- [ ] **All Manim text ≥ font_size 26** — visually inspected after render
- [ ] **No element clips outside the 13.33" × 7.50" canvas** — run boundary check
- [ ] **Content body does not overlap the strip** (strip starts at 6.0")
- [ ] Manim backgrounds match slide background color exactly (#hex match)
- [ ] Finding/Insight strip on every result slide
- [ ] Slide number visible on every slide

### Narration (presenter notes)
- [ ] **Every slide has presenter notes** — no exceptions, including title/section/conclusion
- [ ] Notes were written with the full paper loaded in context
- [ ] Notes were written with ALL prior slides' notes loaded in context
- [ ] Each video slide starts with "[Click to start animation]"
- [ ] Each slide's notes end with a transition sentence to the next slide
- [ ] No finding or number is stated twice across all notes
- [ ] Numbers are written out for speaking ("52 percent" not "52%")
- [ ] Read all notes aloud in sequence — they form a coherent talk without seeing slides
- [ ] No slide exceeds 30 seconds of narration (section headers ≤ 8 seconds)

### Timing
- [ ] Section headers at every major topic transition
- [ ] Total runtime ≤ video limit (test with dry run reading all notes aloud)

---

## 13. Video Embedding Rules (python-pptx)

### CRITICAL: Maintain 16:9 aspect ratio — no exceptions

All Manim animations render at 3840×2160 (16:9). When embedding with `add_movie`, the width and height **must** preserve 16:9, otherwise PowerPoint will non-uniformly stretch the video.

```python
# CORRECT — 16:9 preserved
add_video(slide, "clip.mp4", width=10.0, height=5.625)   # 10 / 5.625 = 1.778 ✓

# WRONG — this stretches the video horizontally
add_video(slide, "clip.mp4", width=12.43, height=4.88)   # 12.43 / 4.88 = 2.547 ✗
```

**Rule: always derive height = width × (9/16), or width = height × (16/9). Never set both independently.**

### Center the video horizontally
```python
VIDEO_W = 10.0
VIDEO_H = VIDEO_W * 9 / 16  # = 5.625
VIDEO_LEFT = (13.33 - VIDEO_W) / 2  # = 1.665
VIDEO_TOP  = 1.05  # just below title
```

### Leave room for captions
Place the Finding/Insight strip at `top=5.8"` (not 6.02"). The 0.75" below the strip (6.65" to 7.50") is reserved for caption/subtitle text — do not place any content there.

---

## 14. Finding/Insight Strip Rules

### One sentence only — no verbatim plot repetition

The strip is NOT a caption and NOT a summary. It is a **punchline** — a single sentence that states the *implication*, not the *numbers*.

```
BAD  — "OLIVE-IE: delta = +0.034 kills/s, p = 0.004; all 8 participants improved, r = +0.15"
GOOD — "EEG-augmented guidance lifts every operator — regardless of how well they shoot."

BAD  — "96.8% guidance convergence in 28.3s vs 68.4% for best baseline"
GOOD — "The agent becomes reliably useful within the first third of every block."
```

**Hard rules:**
- One sentence maximum
- No p-values or delta-values in the strip (those belong in the animation)
- State the *so what*, not the *what*
- 20–24pt font

---

## 15. Section Intro Slides

Use python-pptx directly (no Manim animation) for section transition slides. Keep them text-only:

```python
def section_intro_slide(prs, study_num, question, details, notes=""):
    """
    Large centered study number (olive color) + question + detail line.
    No embedded video — pure text, clean, fast to read.
    """
    slide = prs.slides.add_slide(prs.slide_layouts[6])
    set_bg(slide)
    # Study number — large, olive
    tb1 = slide.shapes.add_textbox(i(0.8), i(1.5), i(11.73), i(1.2))
    p = tb1.text_frame.paragraphs[0]; p.alignment = PP_ALIGN.CENTER
    r = p.add_run(); r.text = study_num
    r.font.size = Pt(28); r.font.bold = True
    r.font.color.rgb = RGBColor(0x6B, 0x8E, 0x23)  # olive
    # Question — white, large
    tb2 = slide.shapes.add_textbox(i(0.8), i(2.5), i(11.73), i(2.2))
    tf = tb2.text_frame; tf.word_wrap = True
    p2 = tf.paragraphs[0]; p2.alignment = PP_ALIGN.CENTER
    r2 = p2.add_run(); r2.text = question
    r2.font.size = Pt(52); r2.font.bold = True; r2.font.color.rgb = WHITE
    # Detail — gray, smaller
    tb3 = slide.shapes.add_textbox(i(0.8), i(5.1), i(11.73), i(0.7))
    p3 = tb3.text_frame.paragraphs[0]; p3.alignment = PP_ALIGN.CENTER
    r3 = p3.add_run(); r3.text = details
    r3.font.size = Pt(26); r3.font.color.rgb = LGRAY
    add_notes(slide, notes)
    return slide
```

---

## 16. Bar Chart Requirements

Every bar chart in a Manim animation must include:
1. **Y-axis tick numbers** — visible, minimum font_size=26
2. **Statistical significance markers** — asterisks or "ns" above each bar
3. **Value labels** on or above each bar
4. The "hero" condition bar must be visually distinct (brighter color + annotation)

```python
# Significance marker above a bar
sig = Text("**", font_size=36, color=GOLD, weight=BOLD)
sig.next_to(bar, UP, buff=0.1)
self.play(FadeIn(sig))
```

---

## 17. When NOT to Show a Chart

Do not include a result panel if it shows no meaningful difference between conditions.
Example: if all four conditions have nearly identical hit rates, showing a hit-rate bar chart wastes screen time and confuses the audience. Show only charts that illustrate a finding.

**Rule: every panel must show a contrast, a trend, or a comparison that supports the narrative.**

