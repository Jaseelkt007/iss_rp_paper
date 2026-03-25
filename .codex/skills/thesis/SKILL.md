---
name: thesis
description: Use when working on the ISS master thesis LaTeX project in this repository, including writing or revising thesis chapters, updating results or figures from evaluation outputs, checking methodology against the codebase, and enforcing thesis-specific academic writing and LaTeX conventions.
metadata:
  short-description: ISS thesis writing and editing guide
---

# Thesis

You are assisting with a Master Thesis written in LaTeX at the Institute for Signal Processing and System Theory (ISS), University of Stuttgart. Apply every rule in this skill precisely whenever editing, adding, or reviewing thesis content.

---

## THESIS IDENTITY

- **Title**: Semantically Controlled Video Generation for Autonomous Driving using Latent Diffusion Models
- **Author**: Mohammed Jaseel Kunnathodika
- **Supervisor**: Khaled Seyam
- **Type**: Master Thesis
- **Institution**: ISS, University of Stuttgart
- **Submission**: 28.02.2026
- **Language**: English (British/American consistent — do not mix)
- **Citation style**: IEEEtran (`\bibliographystyle{IEEEtran_ISS}`) — numbered citations `\cite{key}`
- **LaTeX template**: ISS standard (`scrreprt` class) — do not modify preamble or layout settings

---

## FILE LOCATIONS

| Resource | Path |
|---|---|
| Thesis root | `/usrhomes/s1492/iss_rp_paper/iss_rp_paper/iss-thesis/` |
| Main TeX file | `iss-thesis.tex` |
| Chapter 1 — Introduction | `chapters/ch1_introduction.tex` |
| Chapter 2 — Related Work | `chapters/ch2_related_work.tex` |
| Chapter 3 — Methodology | `chapters/ch3_methodology.tex` |
| Chapter 4 — Experiments | `chapters/ch4_experiments.tex` |
| Chapter 5 — Discussion | `chapters/ch5_discussion.tex` |
| Chapter 6 — Summary & Outlook | `chapters/ch6_summary.tex` |
| Bibliography | `refs.bib` |
| Images (thesis) | `images/` |
| Eval results (source) | `/no_backups/s1492/Ctrl-V/outputs/` |
| Project codebase | `/usrhomes/s1492/Ctrl-V-seg/` |
| Eval skill (metric definitions) | `/usrhomes/s1492/Ctrl-V-seg/.claude/skills/docsum/SKILL.md` |

---

## STEP 1 — Read before writing

Always read the target chapter file before making any edit. Never write from memory alone. If the request involves:
- **Results or comparisons** → read the relevant `eval_results.json` under `/no_backups/s1492/Ctrl-V/outputs/<run>/` before writing any numbers. See the docsum skill for metric definitions and JSON key mappings.
- **Methodology** → read the relevant source files in `/usrhomes/s1492/Ctrl-V-seg/` to verify architectural claims against the actual code before writing. Do not describe what the code does — describe what the method does at the algorithmic level.
- **Figures** → check `images/` for existing files; if a required figure exists in `/no_backups/s1492/Ctrl-V/outputs/` (e.g., confusion matrices, comparison PNGs), copy it to `images/` before referencing it.

---

## STEP 2 — Classify the request

Determine the type of change:
- **New section / subsection**: plan where it fits in the chapter flow, confirm it does not duplicate existing content
- **Update results**: pull fresh numbers from JSON, update tables/text consistently
- **Methodology update**: verify against codebase, check alignment checklist (Step 3)
- **Figure insertion**: check figure exists, copy if needed, write caption, add `\ref` in text
- **Language / writing improvement**: apply all writing rules (Step 4) without changing technical content

---

## STEP 3 — Methodology alignment checklist

When editing Chapter 3 (Methodology) or any section describing the system architecture, verify every claim against the codebase. Work through this checklist before writing:

- [ ] **Stage 1 description correct**: RGB first frame → CLIP embedding (conditioning); semantic maps of all frames encoded via Semantic VAE → noisy latents; UNet (temporal blocks trained) predicts denoised semantic latents; decode → semantic IDs
- [ ] **Stage 2 description correct**: Ground-truth semantic maps → Semantic VAE → scaled latents → ControlNet conditioning; first RGB frame → CLIP + RGB VAE (conditioning); UNet + ControlNet predict denoised RGB latents; decode → RGB video
- [ ] **DualVAEManager described accurately**: manages two VAEs — frozen SVD RGB VAE and frozen pretrained Semantic VAE; encode_semantic_from_ids takes grayscale trainIDs (0–18) → one-hot (19 channels) → 4-channel latents; decode reverses this
- [ ] **Semantic VAE described accurately**: native architecture trained on KITTI-360 semantic maps; encodes discrete 19-class label maps into continuous 4-channel latent representations; jointly trained with dice loss and boundary-aware loss
- [ ] **KITTI-360 classes**: 19 training classes (trainIDs 0–18), consistent with Cityscapes label definition; raw KITTI-360 IDs remapped using official label mapping
- [ ] **Resolution and clip length stated correctly**: training resolution 192×704; latent resolution 24×88 (downsampled by factor 8); clip length 25 frames (SVD-XT)
- [ ] **Scaling factor mentioned if relevant**: latent scaling factor 0.18215 used to normalise VAE latents before diffusion
- [ ] **No implementation-level function names cited**: describe algorithms and components, not code function names, class names, or variable names
- [ ] **No code-level details in methodology**: method descriptions must be algorithm-level (equations, data flow, component roles), not Python implementation details
- [ ] **Stage independence stated**: Stage 1 and Stage 2 are trained independently; Stage 2 uses ground-truth semantic maps, not Stage 1 predictions, during training

---

## STEP 4 — Academic writing rules (apply to every sentence written)

### Tone and register
- Formal, precise, objective throughout
- Never use: "I think", "we believe", "it's great", "amazing", "clearly", "obviously", "basically", "a lot of", "kind of", "sort of"
- Never use first person singular ("I"). Use "we" (standard in academic writing) or passive voice
- No rhetorical questions
- No contractions (use "do not", not "don't")

### Sentence structure
- Prefer short to medium sentences (15–25 words ideal)
- One idea per sentence
- Avoid stacking multiple subordinate clauses
- Begin sentences with the topic, not conjunctions ("However," is acceptable as a transition, "But" is not)

### Paragraph structure (every paragraph must follow this):
1. **Topic sentence** — states the single idea of the paragraph
2. **Supporting explanation** — expands on the topic with reasoning
3. **Evidence / example / equation / citation** — substantiates the claim
4. **Concluding sentence** — connects back to the topic or bridges to the next paragraph

### Plagiarism and citation
- Never copy text from papers, repositories, or documentation verbatim
- Always paraphrase: rewrite the idea in different sentence structure and vocabulary, not just synonym substitution
- Every factual claim about prior work must have a `\cite{}` reference
- Every dataset, architecture, or metric that was introduced elsewhere must be cited on first mention
- Cite using `\cite{key}` inline with the sentence it supports

### Terminology consistency
- Define a term once on first use, then use it consistently throughout
- Key terms and their consistent forms:
  - "latent diffusion model" (not "diffusion model in latent space")
  - "semantic segmentation map" (not "segmentation mask" or "label map" interchangeably)
  - "Semantic Variational Autoencoder" → abbreviated as "Semantic VAE" after first definition
  - "Stable Video Diffusion" → abbreviated as "SVD" after first definition
  - "ControlNet" (one word, capital C and N)
  - "KITTI-360" (hyphenated, always)
  - "mean Intersection-over-Union" → abbreviated as "mIoU"
  - "Fréchet Inception Distance" → abbreviated as "FID"
  - "Fréchet Video Distance" → abbreviated as "FVD"
  - "trainID" (for KITTI-360 remapped class indices 0–18)

### Avoiding redundancy
- Do not restate a definition already given in the same chapter
- Do not repeat the same claim in both the section introduction and the section body
- If a concept was explained in Chapter 2 (Related Work), reference it with `\ref{}` rather than re-explaining

### Figures and tables
- Every figure and table must be referenced in the body text before it appears: "as illustrated in Figure~\ref{fig:xyz}" or "Table~\ref{tab:xyz} presents..."
- Every figure must have a descriptive caption that explains what is shown and what the reader should observe
- Captions end with a period
- Figure captions go below the figure (`\caption{}` after `\includegraphics`)
- Table captions go above the table (`\caption{}` before `\begin{tabular}`)
- Use `\label{}` immediately after `\caption{}` for all floats
- Do not insert a figure without discussing it in the text
- Prefer `[t]` or `[b]` float placement; avoid `[H]` unless a float genuinely must not move
- Use `\FloatBarrier` at section boundaries to prevent floats crossing section headers

### Mathematical notation
- Define every symbol the first time it appears
- Use consistent notation: if $\mathbf{z}$ denotes a latent vector in one equation, do not use $z$ for the same concept later
- Use `\mathbb{R}` for real number spaces (already defined as `\R` in the template)
- Equations that are referred to in text must be numbered: `\begin{equation} ... \label{eq:name} \end{equation}`
- Inline math for simple expressions: $\sigma$, $\mathbf{x}$
- Display math for key equations: `\begin{equation}...\end{equation}`

### Bullet points and lists
- Use `\begin{itemize}` sparingly — only for genuinely enumerable, non-narrative items (e.g., a list of hardware components)
- Never use bullet points to present argument or analysis — write paragraphs instead
- Lists must not replace a paragraph that has logical flow
- Maximum 5 items in any list; if more, convert to a table or prose

### Cross-references and structure
- Use `\ref{ch:...}`, `\ref{sec:...}`, `\ref{fig:...}`, `\ref{tab:...}`, `\ref{eq:...}` — never hard-code numbers
- Label convention already in use: `ch:`, `sec:`, `subsec:`, `fig:`, `tab:`, `eq:`
- Chapter transitions: end each chapter with a brief paragraph summarising what was covered and signposting the next chapter
- Section introductions: begin each section with one sentence stating its scope

### Ethical and scientific integrity
- Report all results honestly, including negative or inconclusive findings
- Do not selectively omit metrics that show poor performance — contextualise them instead
- State dataset and evaluation split clearly (always: KITTI-360, validation split, non-overlapping clips)
- State the number of evaluation samples when reporting any metric
- Do not present the DRN-based mIoU in Stage 2 as a direct measure of generation quality without noting it measures semantic consistency of generated RGB frames as assessed by an external segmentation model

---

## STEP 5 — Handling results and figures

### When updating results
1. Read `/no_backups/s1492/Ctrl-V/outputs/<run>/eval_results.json`
2. Cross-check metric definitions with the docsum skill at `/usrhomes/s1492/Ctrl-V-seg/.claude/skills/docsum/SKILL.md`
3. Report values to two decimal places for percentages (e.g., 40.01\%), four decimal places for FID/FVD/LPIPS
4. Always state: checkpoint step, number of evaluation samples, evaluation split
5. Update both the table and any in-text references to the same number — never leave them inconsistent

### Stage 1 canonical results (from `eval_stage1_semantic/eval_results.json`)
- Checkpoint step: best_checkpoint
- Samples: full clips, 25 frames, 192×704
- Primary metric: mIoU = 40.01%, Pixel Accuracy = 80.84%, Mean Class Accuracy = 53.52%, FW-IoU = 68.75% (examples)

### Stage 2 canonical results (from `eval_stage2_rgb/eval_results.json`)
- Checkpoint step: best_checkpoint
- Samples: full clips, val non-overlapping
- Primary metric: DRN mIoU = 23.20%, Pixel Accuracy = 67.63% (examples)
- Image/video quality: read from JSON (FID, FVD, LPIPS, SSIM, PSNR — check for null values)

### When inserting figures from evaluation outputs
1. Check `/no_backups/s1492/Ctrl-V/outputs/<run>/` for PNG files (confusion matrices, comparison frames, etc.)
2. If the figure is needed in the thesis, copy it:
   ```
   cp /no_backups/s1492/Ctrl-V/outputs/<run>/<file>.png \
      /usrhomes/s1492/iss_rp_paper/iss_rp_paper/iss-thesis/images/<descriptive_name>.png
   ```
3. Use a descriptive filename (e.g., `stage1_confusion_matrix.png`, `stage2_qualitative_comparison.png`)
4. Write a caption that explains: (a) what the figure shows, (b) what the reader should note
5. Reference the figure in the text before it appears

---

## STEP 6 — LaTeX formatting rules

- Use `~` before `\cite`, `\ref`, `\Fig.`, `\Table` to prevent line breaks: `Figure~\ref{fig:x}`, `\cite{key}`
- Use `--` for ranges (en-dash): `0--18`, `192 \times 704`
- Use `\times` for multiplication, not `x`
- Acronyms on first use: "Fréchet Video Distance (FVD)" then `FVD` thereafter
- Use `\emph{}` for emphasis (italics) — sparingly
- Use `\textbf{}` for bold — only in table headers and defined terms on first introduction
- Do not use `\\` for line breaks in paragraph text
- Use `\noindent` only if intentionally suppressing paragraph indent (rare in this template)
- The template uses `parskip=half-` — do not add extra `\vspace` between paragraphs
- Compile check: after editing, verify no undefined references or citations by checking the `.log` file for warnings

---

## STEP 7 — Pre-submission review checklist

Run through this whenever completing a set of edits:

- [ ] All `\ref{}` and `\cite{}` resolve — no `??` in compiled PDF
- [ ] All figures referenced in text before they appear
- [ ] All table captions above, figure captions below
- [ ] No metric value appears in text without matching the table value
- [ ] No function name, class name, or variable name from codebase cited in text
- [ ] No first-person singular ("I")
- [ ] No informal expressions
- [ ] Terminology consistent throughout edited section
- [ ] Each paragraph has a topic sentence, explanation, evidence, and closing
- [ ] Bullet lists not used for analytical content
- [ ] New acronyms defined on first use and added to Appendix A acronym table if not already present
- [ ] Bibliography entry exists for every `\cite{key}` used

---

## STEP 8 — Summary of change made

After completing any edit, report:

```
## Thesis Edit Summary

### Chapter / Section modified
<chapter name, section name, LaTeX label>

### Changes made
- <concise description of what was added/changed/removed>

### Results used (if applicable)
- Source: <eval_results.json path>
- Metrics: <list of values inserted>

### Figures copied (if applicable)
- From: <source path>
- To: images/<filename>
- Caption: <caption text used>

### Writing rules applied
- <any notable rule applied or potential issue flagged>

### Recommended next steps
- <e.g., compile and check for undefined references, add citation for X>
```
