# Figure 3.1 — Two-Stage Pipeline Overview
## Image Generation Prompt

---

## Context

This diagram is **Figure 3.1** in a Master Thesis titled:
> *"Semantically Controlled Video Generation for Autonomous Driving using Latent Diffusion Models"*

The figure illustrates the **complete inference-time pipeline** of the proposed method: a two-stage latent diffusion system that takes a single RGB driving frame as input and outputs a photorealistic video of 25 frames, using semantic segmentation maps as an intermediate structural representation.

The caption in the thesis reads:
> "Overview of the proposed two-stage pipeline for semantically controlled video generation. The Semantic VAE bridges discrete semantic maps and continuous latent space. Stage 1 predicts future semantic sequences. Stage 2 generates photorealistic RGB video conditioned on the predicted semantics."

---

## Visual Style Requirements

- **Style:** Clean, academic technical diagram. No shadows, no decorative 3D effects.
- **Background:** Pure white.
- **Font:** A clean sans-serif font (e.g., Arial, Helvetica, or Inter). All text must be readable at thesis print size (approximately A4 column width, ~8 cm wide).
- **Colour scheme:**
  - **Blue (`#4A90D9` or similar):** Trainable / finetuned components
  - **Gray (`#9E9E9E` or similar):** Frozen pretrained components
  - **Green (`#5CB85C` or similar):** Input and output data tensors / images
  - **Orange (`#E8A838` or similar):** Latent space representations
  - **Purple (`#8E44AD` or similar):** Semantic segmentation maps
  - **Red arrows:** Gradient / training flow (if shown in a training variant)
  - **Black arrows:** Forward inference data flow
- **Arrow style:** Clean directional arrows with filled arrowheads. Label each major arrow with the tensor shape or a short description.
- **Box style:** Rounded rectangles for processing modules. Sharp rectangles or parallelograms for data tensors.

---

## Overall Layout

The diagram should flow **left to right**, structured in **three horizontal rows**:

```
Row 1 (top):     [Shared Input]  →  [CLIP Encoder]  →  [CLIP Embedding c⁽⁰⁾]
Row 2 (middle):  [Stage 1: Semantic Prediction]
Row 3 (bottom):  [Stage 2: Semantic-to-Video Generation]
```

The single RGB input frame at the left feeds into both Stage 1 (top arrow) and Stage 2 (bottom arrow), and the CLIP embedding fans out to cross-attend inside both stages.

---

## Detailed Component Descriptions

### A. Shared Input (leftmost)

**Box: "Initial RGB Frame f⁽⁰⁾"**
- Green box/border
- Show a small thumbnail of a realistic driving scene (road, cars, buildings, sky) to indicate this is a single photograph
- Label: `f⁽⁰⁾ ∈ ℝ^{3×H×W}` below the box

Two arrows emerge from this box:
1. Upward-left → CLIP Encoder
2. Downward into Stage 1 and Stage 2 (branch)

---

### B. CLIP Encoder (top-left)

**Box: "CLIP Vision Encoder"**
- Gray fill (frozen)
- Small lock icon to indicate frozen weights
- Label below: `Frozen`
- Input shape label on arrow: `3 × 224 × 224`
- Output: a vector labeled `c⁽⁰⁾ ∈ ℝ^{1×1024}` (global appearance embedding)

Two dashed arrows from `c⁽⁰⁾` flow downward:
- One into **Stage 1 UNet** (cross-attention pathway)
- One into **Stage 2 UNet and ControlNet** (cross-attention pathway)
Label these dashed arrows: `Cross-attention conditioning`

---

### C. Stage 1 — Semantic Prediction (middle row)

Label the entire row with a titled box or background rectangle:
**"Stage 1: Semantic Video Prediction"** in blue text at top-left of the row.

Sub-components from left to right:

1. **"Semantic VAE Encoder"** (gray box, frozen lock icon)
   - Input: Semantic ID map of first frame `y⁽⁰⁾ ∈ {0,...,18}^{H×W}`
   - Processing: one-hot → Semantic Stem → frozen VAE encoder core
   - Output: `s⁽⁰⁾ ∈ ℝ^{4×h×w}` (purple arrow, first-frame semantic latent)
   - Note: `h = H/8, w = W/8`

2. **"Noise Schedule (EDM)"** (small orange box above the UNet)
   - Adds Gaussian noise to semantic latents: `ŝₜ = αₜ s₀ + σₜ ε`
   - Output: noisy semantic latents (orange, dashed outline)

3. **Channel Concatenation block** (small diamond/junction)
   - Concatenates: `[ŝₜ (4ch) || s⁽⁰⁾ repeated (4ch)]` → 8-channel input
   - Label: `concat → 8ch`

4. **"SVD UNet (temporal blocks fine-tuned)"** (blue box — partially trainable)
   - Large central box in Stage 1
   - Show internal hint: 4 encoder levels + mid block + 4 decoder levels (simple nested boxes)
   - Split fill: gray core (frozen spatial blocks) + blue temporal layers (fine-tuned)
   - Label: `Trainable: temporal transformer blocks`
   - Receives:
     - 8-channel concatenated input from left
     - `c⁽⁰⁾` CLIP embedding via dashed arrow from top (cross-attention)
   - Output: denoised semantic latents `s_{1:T} ∈ ℝ^{T×4×h×w}` (orange arrow)

5. **"Semantic VAE Decoder"** (gray box, frozen lock icon)
   - Input: semantic latents `s_{1:T}`
   - Output: semantic segmentation maps `ŷ_{1:T} ∈ {0,...,18}^{T×H×W}` (purple)
   - Show a small strip of 3–4 thumbnail frames with colour-coded segmentation maps (road=purple, sky=blue, car=red, vegetation=green) to indicate this is a sequence

---

### D. Stage 2 — Semantic-to-Video Generation (bottom row)

Label the row: **"Stage 2: Semantic-to-Video Generation"** in blue text at top-left.

Sub-components from left to right:

1. **"RGB VAE Encoder"** (gray box, frozen lock icon)
   - Input: initial RGB frame `f⁽⁰⁾`
   - Output: `z⁽⁰⁾ ∈ ℝ^{4×h×w}` (orange, initial frame latent)
   - Arrow label: `z⁽⁰⁾ repeated T times`

2. **"Noise Schedule (EDM)"** (small orange box)
   - Adds Gaussian noise to ground-truth RGB latents
   - Output: noisy RGB latents `ẑₜ` (dashed orange)

3. **Channel Concatenation** (junction)
   - Input: `[ẑₜ (4ch) || z⁽⁰⁾ₚₐd (4ch)]` → 8-channel input

4. **"Semantic VAE Encoder"** (gray, frozen)
   - Input: semantic maps from Stage 1 output `ŷ_{1:T}` (purple arrow, indicated as "from Stage 1")
   - Output: semantic latents `s_{1:T} ∈ ℝ^{T×4×h×w}` → feeds into ControlNet

5. **"ControlNet"** (blue box — fully trainable)
   - Input: 8-channel RGB concatenation (same as UNet) **plus** 4-channel semantic latents (additional control input on top)
   - Show a smaller U-Net-like encoder structure
   - Label: `Trainable ControlNet (copy of SVD encoder + mid-block)`
   - Receives `c⁽⁰⁾` via dashed cross-attention arrow from top
   - Outputs: residuals at multiple scales → represented by multiple horizontal arrows pointing into the frozen UNet decoder skip connections
   - Label residual arrows: `residuals r₁, r₂, ..., rₙ` with zero-init markers (small "0" circles on arrows)

6. **"SVD UNet (frozen backbone)"** (gray box, frozen lock icon)
   - Large box
   - Input: 8-channel RGB concatenation
   - Show encoder path (left half of U-Net) + mid block + decoder path (right half)
   - ControlNet residuals injected into decoder skip connections (shown as `+` addition nodes at the skip connections)
   - Receives `c⁽⁰⁾` via dashed cross-attention arrow from top
   - Label: `Frozen SVD backbone`
   - Output: denoised RGB latents `z_{1:T} ∈ ℝ^{T×4×h×w}` (orange)

7. **"RGB VAE Decoder"** (gray box, frozen lock icon)
   - Input: denoised RGB latents `z_{1:T}`
   - Output: final RGB video frames `f̂_{1:T}` (green)
   - Show a strip of 3–4 realistic driving scene thumbnails to indicate photorealistic video output

---

### E. Connection Between Stages (the semantic bridge)

Draw a prominent arrow from the **Stage 1 output** (semantic segmentation maps `ŷ_{1:T}`) flowing **downward** into the **Stage 2 Semantic VAE Encoder** input. This arrow should be thick and purple, labeled:
> `Predicted semantic maps (25 frames)`

This arrow visually represents the sequential dependency at inference time.

Optionally, add a small annotation box near this connection:
> "At training: ground-truth maps used (stages independent)"

---

## Annotations and Labels

Add a **legend box** in a corner of the figure:
| Colour | Meaning |
|--------|---------|
| Blue fill | Trainable / fine-tuned module |
| Gray fill | Frozen pretrained module |
| Orange arrow/tensor | Latent space representation |
| Purple arrow/tensor | Semantic segmentation map |
| Green box | RGB image / video |
| Dashed arrow | Cross-attention conditioning pathway |

Add the following dimension annotations on the diagram:
- Training resolution: `H×W = 192×704`
- Latent resolution: `h×w = 24×88` (downsampled by factor 8)
- Clip length: `T = 25 frames`
- CLIP embedding: `d_c = 1024`
- Semantic classes: `C = 19`

---

## Prompt for AI Image Generator

Use the following as a direct text prompt for an AI diagram generation tool (e.g., GPT-4o, Ideogram, or similar):

---

**PROMPT:**

> Create a clean, white-background academic technical diagram for a machine learning thesis. The diagram shows a two-stage video generation pipeline for autonomous driving.
>
> **Overall layout:** Left-to-right horizontal flow with three rows. Top row: shared CLIP encoder. Middle row: Stage 1 (Semantic Prediction). Bottom row: Stage 2 (Semantic-to-Video Generation).
>
> **Color coding:** Blue rounded rectangles = trainable modules. Gray rounded rectangles with a lock icon = frozen pretrained modules. Orange parallelograms = latent tensors. Purple parallelograms = semantic segmentation maps. Green boxes = RGB images/videos. Black solid arrows = main data flow. Gray dashed arrows = cross-attention conditioning.
>
> **Input (leftmost):** A green box labeled "Initial RGB Frame f⁽⁰⁾" showing a driving scene thumbnail. Two arrows emerge: one to the CLIP encoder and one branching into both stages.
>
> **CLIP Encoder (top center):** Gray box labeled "CLIP Vision Encoder (Frozen)" with a lock icon. Outputs a vector labeled "c⁽⁰⁾ ∈ ℝ^{1×1024}". Two gray dashed arrows flow from this embedding down into both the Stage 1 UNet and the Stage 2 UNet+ControlNet, labeled "Cross-attention conditioning".
>
> **Stage 1 (middle row, labeled "Stage 1: Semantic Video Prediction"):** Left to right: (1) Gray "Semantic VAE Encoder" box receiving semantic IDs and outputting orange latent s⁽⁰⁾. (2) A channel-concatenation junction merging noisy semantic latents (4ch) and the conditioning latent (4ch) into 8 channels. (3) Large blue "SVD UNet (temporal blocks fine-tuned)" U-Net diagram with a split fill showing frozen spatial layers in gray and blue temporal transformer layers. It receives the 8-channel input and the CLIP cross-attention signal, and outputs orange denoised semantic latents s_{1:T}. (4) Gray "Semantic VAE Decoder" box that converts the latents to purple semantic maps — show 4 small color-coded segmentation frame thumbnails.
>
> **Stage 2 (bottom row, labeled "Stage 2: Semantic-to-Video Generation"):** Left to right: (1) Gray "RGB VAE Encoder" receiving f⁽⁰⁾ and outputting orange z⁽⁰⁾. (2) Concatenation junction: noisy RGB latents (4ch) + z⁽⁰⁾ padded (4ch) = 8ch. (3) Gray "Semantic VAE Encoder" receiving purple semantic maps from Stage 1 (prominent purple arrow from above labeled "Predicted semantic maps, 25 frames"), outputting semantic latents into the ControlNet. (4) Blue "ControlNet (trainable)" encoder-only U-Net shape receiving the 8-channel input PLUS the semantic latents on top; outputs multiple orange dashed residual arrows with small "0" circles (zero-init) pointing into the frozen UNet decoder skip connections. (5) Gray "SVD UNet (frozen)" full U-Net shape receiving the 8-channel input, CLIP cross-attention, and ControlNet residuals added at skip connections (shown as "+" nodes). Outputs orange RGB latents z_{1:T}. (6) Gray "RGB VAE Decoder" converting latents to green RGB video — show 4 small photorealistic driving scene frame thumbnails.
>
> **Legend** in bottom-right corner: small color swatches for Blue=trainable, Gray=frozen, Orange=latent, Purple=semantic map, Green=RGB image/video.
>
> Style: No shadows. Clean sans-serif font. Minimal decorative elements. Suitable for printing in a technical thesis at A4 size.

---

## Alternative Structured Description (for draw.io / Lucidchart / Figma)

If creating the figure manually in a diagramming tool, structure it as follows:

### Nodes (boxes)
| ID | Label | Colour | Shape |
|----|-------|--------|-------|
| N1 | Initial RGB Frame f⁽⁰⁾ | Green border | Rectangle |
| N2 | CLIP Vision Encoder (Frozen) | Gray fill | Rounded rect |
| N3 | CLIP Embedding c⁽⁰⁾ | Orange fill | Parallelogram |
| N4 | Stage 1 — Semantic VAE Encoder | Gray fill | Rounded rect |
| N5 | Noise Schedule (EDM) | Light orange | Small rect |
| N6 | Channel Concatenation [8ch] | No fill | Diamond |
| N7 | SVD UNet — Temporal Blocks Fine-tuned | Blue+gray | U-Net shape |
| N8 | Stage 1 — Semantic VAE Decoder | Gray fill | Rounded rect |
| N9 | Predicted Semantic Maps ŷ₁:T | Purple fill | Parallelogram strip |
| N10 | RGB VAE Encoder | Gray fill | Rounded rect |
| N11 | Noise Schedule (EDM) | Light orange | Small rect |
| N12 | Channel Concatenation [8ch] | No fill | Diamond |
| N13 | Stage 2 — Semantic VAE Encoder | Gray fill | Rounded rect |
| N14 | ControlNet (Trainable) | Blue fill | U-Net encoder shape |
| N15 | SVD UNet — Frozen Backbone | Gray fill | U-Net shape |
| N16 | ControlNet Residuals rᵢ | Orange dashed | Multi-arrow |
| N17 | RGB VAE Decoder | Gray fill | Rounded rect |
| N18 | Output RGB Video f̂₁:T | Green fill | Rectangle strip |

### Edges (arrows)
| From | To | Style | Label |
|------|----|-------|-------|
| N1 | N2 | Black solid | RGB frame |
| N2 | N3 | Black solid | 1 × 1024 |
| N3 | N7 | Gray dashed | Cross-attention |
| N3 | N14 | Gray dashed | Cross-attention |
| N3 | N15 | Gray dashed | Cross-attention |
| N1 | N4 | Black solid | y⁽⁰⁾ semantic IDs |
| N4 | N6 | Orange solid | s⁽⁰⁾ ∈ ℝ^{4×h×w} |
| N5 | N6 | Orange dashed | ŝₜ noisy latents |
| N6 | N7 | Orange solid | 8-channel input |
| N7 | N8 | Orange solid | s₁:T latents |
| N8 | N9 | Purple solid | ŷ₁:T maps |
| N9 | N13 | Purple solid, thick | Predicted maps (25 frames) |
| N1 | N10 | Black solid | RGB frame |
| N10 | N12 | Orange solid | z⁽⁰⁾ ∈ ℝ^{4×h×w} |
| N11 | N12 | Orange dashed | ẑₜ noisy RGB latents |
| N12 | N14 | Orange solid | 8-channel input |
| N12 | N15 | Orange solid | 8-channel input |
| N13 | N14 | Orange solid | semantic latents s₁:T |
| N14 | N16 | Orange dashed | Residuals (zero-init) |
| N16 | N15 | Orange dashed | Injected at skip connections |
| N15 | N17 | Orange solid | z₁:T ∈ ℝ^{T×4×h×w} |
| N17 | N18 | Green solid | f̂₁:T RGB video |
