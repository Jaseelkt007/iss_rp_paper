# Figure 3.2 — Semantic-Native VAE Architecture
## Image Generation Prompt

---

## Context

This diagram is **Figure 3.2** in a Master Thesis titled:
> *"Semantically Controlled Video Generation for Autonomous Driving using Latent Diffusion Models"*

The figure shows the internal architecture of the **Semantic-Native Variational Autoencoder (Semantic VAE)** — the component that bridges discrete semantic segmentation maps and the continuous latent space of Stable Video Diffusion (SVD).

The caption in the thesis reads:
> "Architecture of the Semantic-Native VAE. The trainable Semantic Stem and Head (shown in blue) replace the original RGB input/output convolutions and interface directly with the frozen VAE encoder and decoder cores (shown in gray), bypassing the RGB bottleneck."

---

## Key Architectural Facts (must be reflected accurately)

| Component | Type | Trainable | Input Shape | Output Shape |
|-----------|------|-----------|-------------|--------------|
| One-hot encoder | Pre-processing | N/A | `[B·T, H, W]` int (0–18) | `[B·T, 19, H, W]` float |
| Semantic Stem | Conv2D (2 layers) | **Yes (blue)** | `[B·T, 19, H, W]` | `[B·T, 128, H, W]` |
| VAE Encoder Core | 4 down-blocks + mid | **No (gray)** | `[B·T, 128, H, W]` | `[B·T, 4, H/8, W/8]` |
| Latent space | Tensor | N/A | `[B·T, 4, H/8, W/8]` | same |
| VAE Decoder Core | Up-blocks (3D conv) | **No (gray)** | `[B·T, 4, H/8, W/8]` | `[B·T, 128, H, W]` |
| Semantic Head | Conv3D (2 layers) | **Yes (blue)** | `[B·T, 128, H, W]` | `[B·T, 19, H, W]` |
| Argmax | Post-processing | N/A | logits | class IDs 0–18 |

The **Semantic Stem** replaces the original RGB `Conv(3→128)` input projection.
The **Semantic Head** replaces the original RGB `Conv(128→3)` output projection.
The **VAE core** (encoder + decoder) is completely frozen and borrowed from SVD.

---

## Visual Style Requirements

- **Style:** Clean, academic technical diagram. Vertical (top-to-bottom) data flow. No shadows, no decorative effects.
- **Background:** Pure white.
- **Font:** Clean sans-serif (Arial, Helvetica, or Inter). All labels must be legible at A4 print size.
- **Colour scheme:**
  - **Blue (`#4A90D9`):** Trainable components — Semantic Stem and Semantic Head
  - **Gray (`#9E9E9E`):** Frozen pretrained components — VAE Encoder and Decoder cores
  - **Orange (`#E8A838`):** Latent space tensor (4 channels)
  - **Purple (`#8E44AD`):** Semantic segmentation data (19-class)
  - **Red dashed box or strikethrough:** The bypassed / removed original RGB projections (to illustrate what was replaced)
- **Arrows:** Solid downward arrows for main forward pass. Use short labels on every arrow with tensor dimensions.
- **Boxes:** Rounded rectangles for processing modules. Flat rectangles or parallelograms for data tensors.
- **Highlight method:** Add a small lock icon (🔒) next to frozen modules. Add a spark/pencil icon or "Trainable" badge next to the Semantic Stem and Head.

---

## Overall Layout

The diagram flows **top to bottom** in a single vertical column.
The main column shows the forward encoding pass (top half) and decoding pass (bottom half), with the latent space `z` at the centre.

An optional **side annotation column** on the right can show:
- Channel widths at each stage (visual bar showing relative width: 19 → 128 → 4 → 128 → 19)
- Parameter counts: Stem ≈ 115K params, Head ≈ 186K params, Core frozen ≈ 11.3M params

---

## Detailed Component Descriptions (top to bottom)

---

### 1. Input: Semantic ID Map (top)

**Box (purple fill): "Semantic ID Map y"**
- Label: `y ∈ {0, …, 18}^{B·T × H × W}`
- Show a small representative thumbnail: a colour-coded driving-scene segmentation map (road=purple-gray, sky=light blue, car=dark red, vegetation=green, building=brown, person=red, etc. — use Cityscapes colour palette)
- Annotation: `19 semantic classes, KITTI-360 trainIDs`
- Resolution label: `H × W = 192 × 704 (training resolution)`

Arrow down → labeled: `Integer class indices`

---

### 2. One-Hot Encoding (pre-processing step)

**Small box (no fill / white): "One-Hot Encoding"**
- Label: `OneHot: {0,...,18} → {0,1}^C`
- Show the concept: a single integer `k` expanding to a binary vector of length 19 (one `1` at position `k`, rest `0`)
- Output shape: `[B·T, 19, H, W]`

Arrow down → labeled: `19 channels (one per class)`

---

### 3. Semantic Stem — Trainable (blue box)

**Box (blue fill): "Semantic Stem"**
- Badge: **"Trainable"** in white text on a blue ribbon
- Shows two sub-layers (can be nested boxes inside the Stem):
  - Sub-layer 1: `Conv2D(19 → 64, k=3, pad=1)` → `GroupNorm(8)` → `SiLU`
  - Sub-layer 2: `Conv2D(64 → 128, k=3, pad=1)`
- Output shape: `[B·T, 128, H, W]`
- Parameter count annotation: `~115K parameters`

**Side annotation (in red dashed box):** Show the original SVD RGB projection that was replaced:
- `[REMOVED] Conv2D(3 → 128)` — crossed out / gray strikethrough
- Arrow or label: `"Replaces RGB input projection"`

Arrow down (solid, black) → labeled: `128 channels`

---

### 4. Frozen VAE Encoder Core (gray box)

**Box (gray fill, large): "VAE Encoder Core (Frozen)"**
- Lock icon in top-right corner
- Badge: **"Frozen — SVD pretrained weights"**
- Shows internal structure (can be simplified nested boxes):
  - `Down Block 1: 128 → 128 (stride 2, residual + attn)` → spatial: `H/2 × W/2`
  - `Down Block 2: 128 → 256 (stride 2, residual + attn)` → spatial: `H/4 × W/4`
  - `Down Block 3: 256 → 512 (stride 2, residual + attn)` → spatial: `H/8 × W/8`
  - `Mid Block: 512 → 512 (residual + self-attention)`
  - `Output Conv: 512 → 8` → split into `μ` and `log σ²` (4 channels each)
- Output: uses `z = μ` (deterministic, no sampling): `[B·T, 4, H/8, W/8]`
- Annotation: `Latent space aligned with SVD RGB latent geometry`

Arrow down (solid, orange) → labeled: `z = μ ∈ ℝ^{B·T × 4 × H/8 × W/8}`, `(deterministic encoding)`

---

### 5. Latent Space (centre — highlighted)

**Box (orange fill): "Semantic Latent z"**
- Label: `z ∈ ℝ^{B·T × 4 × H/8 × W/8}`
- Resolution annotation: `4 channels, 24 × 88 (at 192×704 input)`
- Annotation: `Compatible with SVD latent space (same geometry)`
- Optional: show a small visual of 4 channel maps as grayscale patches to illustrate the compact representation

This is the **central node** of the diagram. Draw it slightly wider or with a highlighted border to emphasise it is the shared bridge between encoding and decoding.

Arrow down (solid, orange) → labeled: `Latent codes z (scaled by 0.18215 before diffusion)`

---

### 6. Frozen VAE Decoder Core (gray box)

**Box (gray fill, large): "VAE Decoder Core (Frozen)"**
- Lock icon in top-right corner
- Badge: **"Frozen — SVD pretrained weights"**
- Shows internal structure:
  - `Up Block 1: 512 → 256 (stride 2 upsample, 3D conv + attn)` → spatial: `H/4 × W/4`
  - `Up Block 2: 256 → 128 (stride 2 upsample, 3D conv + attn)` → spatial: `H/2 × W/2`
  - `Up Block 3: 128 → 128 (stride 2 upsample, 3D conv + attn)` → spatial: `H × W`
  - `GroupNorm → SiLU → 128-channel features`
- **IMPORTANT:** Show with a horizontal cut/dashed line that features are extracted **before** the original RGB output convolution:
  - Draw the original `Conv(128 → 3)` (RGB output) in a red/gray dashed box labeled `[BYPASSED] RGB output conv`
  - Draw a horizontal arrow branching off **before** this bypassed layer into the Semantic Head
  - Label: `"Features captured before RGB output layer"`
- Output: `[B·T, 128, H, W]` (decoder features, not RGB pixels)

Arrow down (solid, black) → labeled: `128 decoder feature channels`

**Side annotation (in red dashed box):** Show the original SVD RGB projection that was bypassed:
- `[BYPASSED] Conv2D(128 → 3, RGB output)` — crossed out
- Arrow or label: `"Replaces RGB output projection"`

---

### 7. Semantic Head — Trainable (blue box)

**Box (blue fill): "Semantic Head"**
- Badge: **"Trainable"** in white text on a blue ribbon
- Shows two sub-layers:
  - Sub-layer 1: `Conv3D(128 → 64, k=3×3×3, pad=1)` → `GroupNorm(8)` → `SiLU` (temporal context)
  - Sub-layer 2: `Conv3D(64 → 19, k=1×1×1)` (per-pixel class logits)
- Note on Conv3D: `"3D kernels leverage temporal context across frames"`
- Output shape: `[B·T, 19, H, W]` (per-pixel class logits)
- Parameter count annotation: `~186K parameters`

Arrow down → labeled: `Logits ŝ ∈ ℝ^{B·T × 19 × H × W}`

---

### 8. Output: Reconstruction (bottom)

**Box (purple fill): "Reconstructed Semantic Map ŷ"**
- Show: `softmax → argmax → discrete class IDs`
- For training: `cross-entropy loss L_CE` computed against ground truth `y` (show a dashed backward arrow labeled `loss` from output back up to the Stem/Head only, with a note "gradients flow through frozen core but only Stem+Head weights update")
- For inference: `argmax → ŷ ∈ {0,...,18}^{B·T × H × W}`
- Show the same small driving-scene segmentation thumbnail as the input (ideally with high fidelity to indicate accurate reconstruction)

---

## Side Panel: Channel Width Visualisation

On the right side of the diagram, draw a horizontal bar chart showing the number of channels at each stage:

```
Stage             | Channels  | Bar width (proportional)
Input (one-hot)   | 19        | ██
Semantic Stem out | 128       | ████████████
VAE Encoder in    | 128       | ████████████
Latent z          | 4         | ▌
VAE Decoder out   | 128       | ████████████
Semantic Head in  | 64 → 19   | ██████ → ██
```

This helps the reader understand the information bottleneck: 19 classes → 128 → **4** (latent) → 128 → 19.

---

## Key Insight Annotations

Add three annotation callout boxes alongside the diagram:

1. **Near the Semantic Stem (top-right):**
   > "Replaces RGB-specific input projection. Avoids 19→3 information bottleneck."

2. **Near the Latent Space (centre-right):**
   > "Latent space shared with SVD RGB VAE. Enables semantic and RGB diffusion in the same geometric space."

3. **Near the Semantic Head (bottom-right):**
   > "3D convolutions couple temporal frames for temporally consistent class predictions."

---

## Prompt for AI Image Generator

Use the following as a direct text prompt for an AI diagram generation tool:

---

**PROMPT:**

> Create a clean, white-background academic technical diagram for a machine learning thesis. The diagram shows the architecture of a "Semantic-Native VAE" — a Variational Autoencoder adapted to encode and decode semantic segmentation maps instead of RGB images.
>
> **Layout:** Vertical, top-to-bottom data flow. A single central column with labeled arrows between boxes.
>
> **Color coding:** Blue boxes = trainable components. Gray boxes with a lock icon = frozen pretrained components. Orange parallelogram = latent tensor (4 channels). Purple boxes = semantic segmentation data. Red dashed/strikethrough = original RGB layers that were removed or bypassed.
>
> **Top:** A purple box labeled "Semantic ID Map y ∈ {0,...,18}^{B·T×H×W}" with a small thumbnail of a colour-coded driving-scene segmentation map next to it.
>
> **Step 1:** A small white box "One-Hot Encoding" converting integer IDs to a 19-channel binary tensor [B·T, 19, H, W].
>
> **Step 2:** A blue rounded rectangle labeled "Semantic Stem (Trainable)" with internal detail showing two Conv2D layers: "Conv2D(19→64) + GroupNorm + SiLU" and "Conv2D(64→128)". Badge: "~115K parameters". On the right side, a red dashed box crossed out: "[REMOVED] Conv2D(3→128) — original RGB input projection". Output shape: [B·T, 128, H, W].
>
> **Step 3:** A large gray rounded rectangle labeled "VAE Encoder Core (Frozen)" with a lock icon. Shows 3 down-sampling blocks reducing spatial size from H×W to H/8×W/8, and a mid-block. Final output: mean μ of Gaussian posterior. Output shape: [B·T, 4, H/8, W/8].
>
> **Centre:** An orange parallelogram labeled "Semantic Latent z = μ ∈ ℝ^{4 × H/8 × W/8}" with annotation "24×88 at 192×704 resolution. Compatible with SVD latent geometry." This is the central node — draw it slightly wider with an orange glow or border.
>
> **Step 4:** A large gray rounded rectangle labeled "VAE Decoder Core (Frozen)" with a lock icon. Shows 3 up-sampling blocks. Importantly, draw a horizontal dashed cut-line near the bottom of this box with a branch arrow going right into the Semantic Head, labeled "features extracted before RGB output layer". Below the cut, draw a small gray strikethrough box: "[BYPASSED] Conv2D(128→3) — original RGB output projection".
>
> **Step 5:** A blue rounded rectangle labeled "Semantic Head (Trainable)" showing two Conv3D layers: "Conv3D(128→64, 3×3×3) + GroupNorm + SiLU" and "Conv3D(64→19, 1×1×1)". Badge: "~186K parameters". Note: "3D kernels use temporal context". Output: class logits [B·T, 19, H, W].
>
> **Bottom:** A purple box labeled "Reconstructed Semantic Map ŷ" showing argmax over 19 channels. A dashed backward arrow from this box going all the way up to the Stem and Head only (not the frozen core), labeled "cross-entropy loss L_CE — only Stem+Head weights updated".
>
> **Right panel:** A vertical bar chart showing channel widths at each stage: 19 → 128 → 4 (bottleneck) → 128 → 19. Annotate with "information bottleneck: 4 channels" at the narrowest point.
>
> **Three callout annotations** alongside the diagram explaining: (1) the Stem removes the 19→3 bottleneck; (2) the latent space is shared with SVD; (3) 3D head convolutions enforce temporal consistency.
>
> Style: No shadows. Clean academic sans-serif font. Suitable for printing in a technical thesis at A4 column width (~8 cm wide).

---

## Alternative Structured Description (for draw.io / Lucidchart / Figma)

### Nodes
| ID | Label | Colour | Shape |
|----|-------|--------|-------|
| N1 | Semantic ID Map y | Purple fill | Rectangle |
| N2 | One-Hot Encoding | White fill | Rounded rect (small) |
| N3 | Semantic Stem (Trainable) | Blue fill | Rounded rect (tall) |
| N3a | [REMOVED] Conv2D(3→128) | Red border, strikethrough | Small rect, right side |
| N4 | VAE Encoder Core (Frozen) | Gray fill | Rounded rect (tall) |
| N5 | Semantic Latent z | Orange fill | Parallelogram |
| N6 | VAE Decoder Core (Frozen) | Gray fill | Rounded rect (tall) |
| N6a | [BYPASSED] Conv2D(128→3) | Red border, strikethrough | Small rect inside N6, bottom |
| N7 | Semantic Head (Trainable) | Blue fill | Rounded rect (tall) |
| N8 | Reconstructed Semantic Map ŷ | Purple fill | Rectangle |

### Edges
| From | To | Style | Label |
|------|----|-------|-------|
| N1 | N2 | Black solid ↓ | integer class IDs |
| N2 | N3 | Black solid ↓ | [B·T, 19, H, W] |
| N3 | N4 | Black solid ↓ | [B·T, 128, H, W] |
| N4 | N5 | Orange solid ↓ | z = μ ∈ ℝ^{4 × H/8 × W/8} |
| N5 | N6 | Orange solid ↓ | latent codes |
| N6 | N7 | Black solid ↓ (branches before N6a) | [B·T, 128, H, W] (before RGB layer) |
| N7 | N8 | Purple solid ↓ | logits → argmax |
| N8 | N3 | Black dashed ↑ | L_CE loss (Stem+Head only) |
| N3a | N3 | Red arrow | "replaces" |
| N6a | N6 | Red annotation | "bypassed" |
