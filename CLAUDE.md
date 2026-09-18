# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A collection of independent LaTeX lecture-note / self-study documents. There is no
build script, no Makefile, and no shared root preamble — each top-level
directory is a self-contained document with its own preamble and its own macro set.

`README.md` is the public-facing showcase (the repo is linked from the author's
LinkedIn profile); `assets/` holds the page-preview PNGs it embeds. See
"Keeping the README in sync" below.

## Building

Always compile from *inside* the document's directory — every main file uses
relative `\input{sections/...}` paths, and running `pdflatex` from the repo root
fails with "Emergency stop / file error" and leaves a stray `texput.log` at the
root (gitignored; delete it, it is not a build product of any document).

```bash
cd multivariate_methods && latexmk -pdf multivariate_methods.tex   # resolves TOC/refs in one go
cd multivariate_methods && pdflatex -interaction=nonstopmode multivariate_methods.tex  # single pass
```

Two passes (or `latexmk`) are required whenever the TOC, `\label`/`\ref`, or
equation numbering changed — the documents put `\tableofcontents` inside a
`\twocolumn[...]` block, so a stale `.toc` silently misprints the front matter.

If `latexmk` reports "Nothing to do" after you edited only a `sections/*.tex`
file, force a rebuild with `latexmk -pdf -g`.

To inspect output without a PDF viewer: `pdftotext multivariate_methods.pdf -`.
Toolchain is TeX Live 2025 (`pdflatex`, `latexmk`, `pdftotext` all on PATH).

Build artifacts (`.aux .log .fls .fdb_latexmk .synctex.gz .toc .out`) are ignored
by the root `.gitignore`. The compiled PDFs are **tracked on purpose** (since
`35435f9`) so they can be read on GitHub — after editing any `.tex`, rebuild and
commit the PDF alongside the source, otherwise the published PDF silently lags.
The `!**/resources/*.pdf` line in `.gitignore` is vestigial: no `resources/`
directory exists any more.

## Documents

| Directory | Main file | Class / style |
|---|---|---|
| `multivariate_methods/` | `multivariate_methods.tex` | dense two-column `article`, five `sections/*.tex` |
| `math_self_study/` | `self_study.tex` | dense two-column `article`, three `sections/*.tex` |
| `mathematical_methods/` | `AMMF.tex` | dense two-column `article`, **Spanish**, twelve `sections/*.tex` |
| `razonamiento_incertidumbre/` | `razonamiento_incertidumbre.tex` | dense two-column `article`, **Spanish**, two `sections/*.tex` |

`math_self_study/` is still **untracked** in git (it has never been committed);
`git add` it deliberately when it is ready to publish, since the README already
links to its PDF.

`mathematical_methods/differential_geometry.tex` is **not a chapter** despite
the name — it is a *preamble fragment* (no `\documentclass`, no
`\begin{document}`) of the boxed-theorem family, and it is **orphaned**:
`AMMF.tex` was rewritten onto the dense two-column family and no longer
`\input`s it. The `cryptography/` directory that once held a twin copy has been
removed, so this file is now the only surviving example of that family. It can
be deleted, or reused as the base of a new boxed-style document.

## The two preamble families

**Dense two-column family** (`multivariate_methods`, `math_self_study`,
`mathematical_methods`, `razonamiento_incertidumbre`) — 10pt,
`twocolumn`, `a4paper`, tight `geometry`, `titlesec`/`enumitem`/`tocloft` for
compact headings and lists, `fancyhdr` running head, `\numberwithin{equation}{subsection}`.
Theorem-likes are **hand-rolled** with `\newenvironment` over a single shared
`thm` counter reset per section — deliberately plain: no `amsthm` styles, no
boxes, no color. All take an optional name argument:

```latex
\begin{definition}[One-to-one correspondence] ... \end{definition}
\begin{theorem}[Spectral decomposition] ... \end{theorem}   % body auto-italic
\begin{proof} ... \end{proof}                                % auto \square
```

Available: `theorem`, `lemma`, `corollary`, `proposition`, `definition`,
`example`, `remark` (unnumbered), `proof`.

`mathematical_methods/AMMF.tex` is the **Spanish variant** of this family and
differs in four ways that matter when copying content in or out:

- environment names are Spanish — `teorema`, `lema`, `corolario`,
  `proposicion`, `definicion`, `ejemplo`, `observacion` (unnumbered),
  `demostracion` (auto `\square`) — and `amsthm` is *not* loaded, so
  `demostracion` is a plain `\newenvironment`, not a `\renewenvironment`;
- babel is `[english,spanish,es-noshorthands,es-nodecimaldot,es-nolists]`. All
  three modifiers are load-bearing: without them `"` becomes active, `,` turns
  into a decimal comma **inside math mode**, and babel fights `enumitem` over
  list spacing;
- accented characters are written as escapes (`\'a`, `\~n`) as elsewhere in the
  repo, even though `inputenc` is UTF-8;
- `\tableofcontents` sits *outside* the `\twocolumn[...]` block. That optional
  argument is a single unbreakable one-column box, and this document's TOC is
  too long to fit on one page — leaving it inside makes it overflow silently
  onto the running text. Do not move it back.

Extra macros here that the other two lack: `\Ext{n}` (the space of `n`-forms
`\bigwedge^n L`), `\rot`, `\Real`, `\Imag`, `\Argu`, `\braket` (inner product
of functions), `\lito` (Landau little-o), `\dnv` (n-th derivative), `\Ee`,
`\dg`. It does *not* define `\E`, `\Var`, `\normal` or the bold-Greek
statistics macros of `multivariate_methods`.

`razonamiento_incertidumbre/razonamiento_incertidumbre.tex` is the **second
Spanish variant** of this family. It uses the same Spanish environment names as
`AMMF.tex` and likewise keeps `\tableofcontents` outside the `\twocolumn[...]`
block, and adds one environment the others do not have — `supuesto` (numbered
off the same `thm` counter, used for the Markov/stationarity/sensor
assumptions). Its own macros are probabilistic: `\Prob` (bold — a *distribution*
over a variable's domain) vs `\prob` (plain — a single number), `\given`,
`\argmax`/`\argmin`, the state/evidence shorthands `\X{k}`, `\Xr{m}{n}`,
`\Ev{k}`, `\Evr{m}{n}`, `\ev{k}`, `\evr{m}{n}`, plus `\Tm`, `\Verd`, `\Fal`.
It carries none of the vector-calculus or complex-analysis macros of `AMMF.tex`
(`\Ext`, `\rot`, `\Real`, `\braket`, …). Its TikZ load is
`arrows.meta, positioning, fit, calc, shapes.geometric` — `shapes.geometric` is
required for the `ellipse` node shape of the dynamic Bayesian network figure,
and omitting it fails with a `pgfkeys` "I do not know the key '/tikz/ellipse'"
error, not a missing-shape one. The DBN figure is a `figure*` (spans both
columns).

**Boxed-theorem family** (only the orphaned
`mathematical_methods/differential_geometry.tex`) — `report`
class, `thmtools` + `mdframed` (`framemethod=TikZ`) with named styles
(`thmgreenbox`, `thmredbox`, `thmbluebox`, `thmblueline`, `thmproofbox`,
`thmexplanationbox`), definitions numbered within chapter and everything else
`sibling=definition`. Extra unnumbered environments here: `uovt`, `notation`,
`previouslyseen`, `problem`, `observe`, `property`, `intuition`. Do not mix
constructs between the two families — `\declaretheorem` styles, `tikz-cd`, and
`\hr` exist only in this one; the dense family's compact-TOC and `\twocolumn`
front matter exist only in the other.

## Macros

Every macro is defined in the main file's preamble, so `sections/*.tex` are only
loadable in their own parent document. The dense family shares a common core —
Greek shorthands (`\al \be \ga \de \ep \la \si \Si \om \ka \vph`), calculus
(`\dd \dv \ddv \pdv \pddv \inte`), linear algebra (`\vb \vh \mat \tr \diag
\rank \spn \adj \inner \norm \abs \tp`), sets (`\R \Z \N \Q \C \F \such \imp`),
operators (`\grad \dive \curl \Lap`), and sizing (`\qty \sqty \set \eval`).

The two preambles have **drifted**, and this is the main hazard when moving
content between them. `multivariate_methods` additionally has:

- `inputenc`/`fontenc`/`babel[spanish,english]`, `amsthm`, `float`, TikZ decorations
- `\inv`, `\I`, statistics macros `\E \Var \Cov \Corr \normal`
- bold-Greek parameter macros `\mub \epsb \Psib \rhob`
- `proof` defined with `\renewenvironment` (because `amsthm` already provides it)

`math_self_study` lacks all of the above and defines `proof` with
`\newenvironment` instead. Copying a `multivariate_methods` section into
`self_study.tex` will fail on undefined `\mub`, `\E`, `\tp`-adjacent stats macros;
copying the other way will fail on the duplicate `proof` definition. Add the
missing macro to the target preamble rather than rewriting the math.

## Conventions

- Sections carry `\label{sec:...}`, `\label{subsec:...}`, `\label{subsubsec:...}`;
  numbered equations carry `\label{eq:...}`. Follow the existing snake_case names.
- Section bodies in `math_self_study` are indented two spaces under their
  `\section`; `multivariate_methods` bodies are flush left. Match the file you edit.
- `multivariate_methods` is intentionally **bilingual**: derivations in English,
  procedural summaries (*resumen del proceso*, *planteamiento*) in Spanish, per
  `sections/notation.tex`. Preserve that split; do not translate Spanish blocks.
- `sections/notation.tex` is the authority for symbols in that document
  (`\vb{X}`, `\mub`, `\Si`, `\tp`) — read it before adding new notation there.
- `math_self_study/sections/number_theory.tex` is a stub (`\section{Number Theory}`
  only) and is already `\input`; content goes into that file.
- VS Code spell/grammar check is set to `en-US` (`math_self_study/.vscode/settings.json`),
  which flags the Spanish passages as errors — expected, not a defect.

## Keeping the README in sync

`README.md` hard-codes facts that drift as the notes grow. When a document or
section is added, update:

- the **Documents** table (page count, topics) and the `typeset pages` badge
  (sum of all PDF page counts; `pdfinfo x.pdf | grep Pages`);
- the per-document bullet lists under **What's inside** and the section counts
  in the headings and in the **Layout** tree;
- the preview PNGs in `assets/` if the showcased page moved. They are single
  pages rendered at 110 dpi with poppler, e.g.
  `pdftoppm -png -r 110 -f 8 -l 8 multivariate_methods/multivariate_methods.pdf assets/preview_multivariate_methods`
  (poppler appends `-8`/`-08` to the name; rename afterwards). Pick a page with
  equations and a figure, not a TOC page.

The README is written for recruiters and fellow students, in English, and
deliberately says nothing about macro drift, preamble families or build
pitfalls — that is what this file is for.
