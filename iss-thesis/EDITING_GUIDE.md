# Thesis Editing Guide

## Table of Contents
1. [Adding Pictures/Figures](#adding-pictures)
2. [Editing Tables](#editing-tables)
3. [Compiling the Thesis](#compiling)
4. [Common LaTeX Tips](#common-tips)

---

## Adding Pictures/Figures {#adding-pictures}

### Basic Figure Syntax

```latex
\begin{figure}[t]  % [t]=top, [h]=here, [b]=bottom, [p]=separate page
    \centering
    \includegraphics[width=0.8\textwidth]{path/to/image.png}
    \caption{Your caption text here}
    \label{fig:your_label}
\end{figure}
```

### Sizing Options

#### 1. **Width-based (Recommended)**
```latex
% Relative to text width (most common)
\includegraphics[width=0.5\textwidth]{image.png}   % 50% of text width
\includegraphics[width=0.8\textwidth]{image.png}   % 80% of text width
\includegraphics[width=\textwidth]{image.png}      % 100% of text width

% Relative to line width (same as textwidth in single column)
\includegraphics[width=0.6\linewidth]{image.png}
```

#### 2. **Height-based**
```latex
\includegraphics[height=5cm]{image.png}            % Fixed height
\includegraphics[height=0.3\textheight]{image.png} % 30% of page height
```

#### 3. **Scale-based**
```latex
\includegraphics[scale=0.5]{image.png}  % 50% of original size
\includegraphics[scale=1.5]{image.png}  % 150% of original size
```

#### 4. **Combined (maintains aspect ratio)**
```latex
% Width constraint with max height
\includegraphics[width=0.8\textwidth, height=0.4\textheight, keepaspectratio]{image.png}
```

### Auto-Rescaling
LaTeX automatically maintains aspect ratio unless you specify both width AND height without `keepaspectratio`. 

**Best practice:** Use `width=X\textwidth` only — LaTeX will auto-scale height proportionally.

### Where to Put Image Files

Create an `images/` or `figures/` folder in the thesis directory:
```
iss-thesis/
├── iss-thesis.tex
├── chapters/
├── images/          ← Put your images here
│   ├── pipeline.png
│   ├── vae_arch.pdf
│   └── results/
│       ├── qualitative1.png
│       └── qualitative2.png
└── refs.bib
```

Then reference them:
```latex
\includegraphics[width=0.9\textwidth]{images/pipeline.png}
\includegraphics[width=0.7\textwidth]{images/results/qualitative1.png}
```

### Supported Image Formats
- **PDF** (vector, best for diagrams/plots) — Recommended!
- **PNG** (raster, good for screenshots/photos)
- **JPG** (raster, good for photos)
- **EPS** (vector, older format)

**Tip:** Use PDF for diagrams created in tools like Inkscape, draw.io, or matplotlib. Use PNG for screenshots.

### Example: Replace Placeholder in Chapter 3

**Current placeholder:**
```latex
\begin{figure}[t]
    \centering
    \fbox{\parbox{0.9\textwidth}{\centering\vspace{3cm}\textbf{[Pipeline Overview Diagram]}...}}
    \caption{Overview of the proposed two-stage pipeline...}
    \label{fig:pipeline_overview}
\end{figure}
```

**Replace with your image:**
```latex
\begin{figure}[t]
    \centering
    \includegraphics[width=0.9\textwidth]{images/pipeline_diagram.pdf}
    \caption{Overview of the proposed two-stage pipeline for semantically controlled video generation. The Semantic VAE bridges discrete semantic maps and continuous latent space. Stage~1 predicts future semantic sequences. Stage~2 generates photorealistic RGB video conditioned on the predicted semantics.}
    \label{fig:pipeline_overview}
\end{figure}
```

### Multiple Images Side-by-Side (Subfigures)

```latex
\begin{figure}[t]
    \centering
    \begin{subfigure}[b]{0.48\textwidth}
        \centering
        \includegraphics[width=\textwidth]{images/input.png}
        \caption{Input frame}
        \label{fig:input}
    \end{subfigure}
    \hfill
    \begin{subfigure}[b]{0.48\textwidth}
        \centering
        \includegraphics[width=\textwidth]{images/output.png}
        \caption{Generated output}
        \label{fig:output}
    \end{subfigure}
    \caption{Comparison of input and generated frames}
    \label{fig:comparison}
\end{figure}
```

### Grid of Images (2x2 example)

```latex
\begin{figure}[t]
    \centering
    \begin{subfigure}[b]{0.48\textwidth}
        \includegraphics[width=\textwidth]{images/frame1.png}
        \caption{Frame 1}
    \end{subfigure}
    \hfill
    \begin{subfigure}[b]{0.48\textwidth}
        \includegraphics[width=\textwidth]{images/frame2.png}
        \caption{Frame 2}
    \end{subfigure}
    
    \vspace{0.5cm}
    
    \begin{subfigure}[b]{0.48\textwidth}
        \includegraphics[width=\textwidth]{images/frame3.png}
        \caption{Frame 3}
    \end{subfigure}
    \hfill
    \begin{subfigure}[b]{0.48\textwidth}
        \includegraphics[width=\textwidth]{images/frame4.png}
        \caption{Frame 4}
    \end{subfigure}
    \caption{Generated video frames}
    \label{fig:video_frames}
\end{figure}
```

---

## Editing Tables {#editing-tables}

### Basic Table Structure

```latex
\begin{table}[t]
    \centering
    \caption{Your table caption (goes ABOVE the table)}
    \label{tab:your_label}
    \begin{tabular}{lcc}  % l=left, c=center, r=right aligned columns
        \toprule
        \textbf{Column 1} & \textbf{Column 2} & \textbf{Column 3} \\
        \midrule
        Row 1 data & 123 & 45.6 \\
        Row 2 data & 789 & 12.3 \\
        \bottomrule
    \end{tabular}
\end{table}
```

### Column Alignment Options

```latex
\begin{tabular}{lcr}     % Left, Center, Right
\begin{tabular}{|l|c|r|} % With vertical lines (not recommended with booktabs)
\begin{tabular}{p{3cm}cc} % Fixed width column (3cm), auto-wrapping
```

### Example: Update Table 5 (Video Metrics)

**Current table in `chapters/ch4_experiments.tex` around line 250:**
```latex
\begin{table}[t]
    \centering
    \caption{Quantitative comparison... \textit{Values marked with $^*$ are placeholder...}}
    \label{tab:video_metrics}
    \begin{tabular}{lccccc}
        \toprule
        \textbf{Method} & \textbf{FID $\downarrow$} & \textbf{FVD $\downarrow$} & ... \\
        \midrule
        SVD (unconditioned) & 85.2$^*$ & 890.5$^*$ & ... \\
        Ctrl-V (bbox control) & 42.1$^*$ & 510.3$^*$ & ... \\
        ...
```

**To update values:**
1. Open `chapters/ch4_experiments.tex`
2. Find the table (search for `tab:video_metrics`)
3. Replace `85.2$^*$` with your actual value, e.g., `87.3`
4. Remove the `$^*$` marker once you have real data
5. Update the caption to remove the placeholder note

**Example updated row:**
```latex
SVD (unconditioned) & 87.3 & 912.4 & 0.54 & 0.26 & 10.2 \\
```

### Adding/Removing Rows

**Add a row:** Just add a new line before `\bottomrule`:
```latex
        Existing row & 1 & 2 & 3 \\
        New row      & 4 & 5 & 6 \\  % ← New row added
        \bottomrule
```

**Remove a row:** Delete the entire line including `\\`

### Adding/Removing Columns

**Add a column:**
1. Update column spec: `{lcc}` → `{lccc}` (add one more `c`)
2. Add header: `\textbf{Col1} & \textbf{Col2} & \textbf{NewCol} \\`
3. Add data to each row: `Data1 & Data2 & NewData \\`

**Remove a column:**
1. Update column spec: `{lccc}` → `{lcc}`
2. Remove from header and all data rows

### Formatting Numbers

```latex
% Bold
\textbf{35.74}

% Italics
\textit{placeholder}

% Arrows (requires amsmath)
\textbf{FID $\downarrow$}  % Down arrow (lower is better)
\textbf{SSIM $\uparrow$}   % Up arrow (higher is better)

% Scientific notation
$3.5 \times 10^{-4}$

% Percentage
89.7\%
```

### Multi-row/Multi-column Cells

```latex
% Requires: \usepackage{multirow}

% Spanning 2 columns
\multicolumn{2}{c}{Spanning text} & Normal cell \\

% Spanning 2 rows
\multirow{2}{*}{Spanning text} & Row 1 data \\
                               & Row 2 data \\
```

### Table Width Control

```latex
% Fixed total width
\begin{tabular*}{\textwidth}{@{\extracolsep{\fill}}lcc}
    ...
\end{tabular*}

% Or use tabularx for auto-sizing columns
\begin{tabularx}{\textwidth}{lXX}  % X columns auto-expand
    ...
\end{tabularx}
```

---

## Compiling the Thesis {#compiling}

### Required Commands (in order)

```bash
cd /usrhomes/s1492/iss_rp_paper/iss_rp_paper/iss-thesis/

# First pass - generates .aux file
pdflatex iss-thesis.tex

# Process bibliography
bibtex iss-thesis

# Second pass - includes citations
pdflatex iss-thesis.tex

# Third pass - resolves all references
pdflatex iss-thesis.tex
```

### Why Multiple Passes?
- **Pass 1:** Generates auxiliary files, collects labels
- **BibTeX:** Processes citations from `refs.bib`
- **Pass 2:** Includes bibliography, updates references
- **Pass 3:** Resolves any remaining cross-references

### Quick Compile (after first full compile)

If you only changed text (no new citations/labels):
```bash
pdflatex iss-thesis.tex
```

### Viewing Output

The compiled PDF will be: `iss-thesis.pdf`

### Common Compilation Errors

**Error: "Undefined control sequence"**
- Missing package or custom command
- Check if you need to add `\usepackage{...}`

**Error: "File not found"**
- Image path is wrong
- Make sure image file exists in the specified location

**Error: "Missing $ inserted"**
- Math mode issue
- Use `$...$` for inline math, `\[...\]` or `\begin{equation}...\end{equation}` for display math

**Warning: "Reference undefined"**
- Need another compile pass
- Or you forgot to add `\label{...}` to the referenced item

---

## Common LaTeX Tips {#common-tips}

### Referencing Figures and Tables

```latex
% In text:
As shown in Figure~\ref{fig:pipeline_overview}, the pipeline consists of...
Table~\ref{tab:video_metrics} presents the quantitative results...

% The ~ creates a non-breaking space (keeps "Figure" and number together)
```

### Math Mode

```latex
% Inline math
The loss is $\mathcal{L} = \alpha + \beta$.

% Display math (numbered)
\begin{equation}
    \mathcal{L}_{\text{total}} = \mathcal{L}_{\text{CE}} + \lambda \mathcal{L}_{\text{Dice}}
    \label{eq:total_loss}
\end{equation}

% Display math (unnumbered)
\[
    \text{IoU} = \frac{TP}{TP + FP + FN}
\]

% Multi-line aligned equations
\begin{align}
    x &= a + b \\
    y &= c + d
\end{align}
```

### Special Characters

```latex
% Percent
89.7\%

% Quotes
``double quotes'' or `single quotes'

% Dashes
- hyphen
-- en-dash (ranges: 1--10)
--- em-dash (punctuation)

% Tilde (non-breaking space)
Figure~\ref{fig:example}

% Backslash in text
\textbackslash
```

### Lists

```latex
% Bulleted list
\begin{itemize}
    \item First item
    \item Second item
\end{itemize}

% Numbered list
\begin{enumerate}
    \item First item
    \item Second item
\end{enumerate}

% Description list
\begin{description}
    \item[Term 1] Definition 1
    \item[Term 2] Definition 2
\end{description}
```

### Text Formatting

```latex
\textbf{bold text}
\textit{italic text}
\texttt{monospace/code text}
\underline{underlined text}
\emph{emphasized text}
```

### Citations

```latex
% In text
As shown by Smith et al.~\cite{smith2020paper}, ...

% Multiple citations
Recent work~\cite{paper1, paper2, paper3} has shown...

% Citation in parentheses (already done by \cite)
This approach~\cite{ho2020denoising} is widely used.
```

### Commenting Out Text

```latex
% Single line comment

\iffalse
Multi-line comment
that spans
several lines
\fi
```

---

## Summary of Changes Made

### Abstract Fix
✅ **Abstract now appears on separate pages:**
- English abstract on one page
- German abstract (Kurzfassung) on another page
- Both appear BEFORE table of contents
- Keywords removed from title page
- Abstract simplified to high-level overview (not implementation details)

### What You Need to Do

1. **Add your images** to `iss-thesis/images/` folder
2. **Replace placeholder figures** in chapter files (search for `\fbox{\parbox`)
3. **Update placeholder table values** (search for `$^*$` in `ch4_experiments.tex`)
4. **Compile** using the commands above
5. **Review** the PDF output

### Quick Reference: Files to Edit

- **Add images:** Create `images/` folder, put files there
- **Replace figure placeholders:** `chapters/ch2_related_work.tex`, `chapters/ch3_methodology.tex`, `chapters/ch4_experiments.tex`
- **Update table values:** `chapters/ch4_experiments.tex` (search for `tab:video_metrics`, `tab:per_class_iou`)
- **Adjust text:** Any chapter file in `chapters/`

---

## Need Help?

If you encounter issues:
1. Check the LaTeX log file (`iss-thesis.log`) for detailed error messages
2. Search for the error message online (LaTeX errors are well-documented)
3. Make sure all image files exist in the correct paths
4. Verify all `\begin{...}` have matching `\end{...}`
