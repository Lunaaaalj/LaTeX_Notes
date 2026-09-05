# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A collection of independent LaTeX lecture-note / self-study documents. There is no
build script, no Makefile, and no shared root preamble — each top-level
directory is a self-contained document with its own preamble and its own macro set.

## Building

Always compile from *inside* the document's directory — every main file uses
relative `\input{sections/...}` paths, and running `pdflatex` from the repo root
fails with "Emergency stop / file error" (this is what `texput.log` at the root is a
leftover of; it is a stray artifact, not a build product of any document).

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

Build artifacts (`.aux .log .fls .fdb_latexmk .synctex.gz .toc .out`) and the compiled
PDFs are ignored by the root `.gitignore`; reference material under `resources/` is
explicitly un-ignored because it is a source asset, not output.

## Documents

| Directory | Main file | Class / style |
|---|---|---|
| `multivariate_methods/` | `multivariate_methods.tex` | dense two-column `article`, four `sections/*.tex` |
| `math_self_study/` | `self_study.tex` | dense two-column `article`, three `sections/*.tex` |
| `mathematical_methods/` | `AMMF.tex` | `report` + boxed theorems |
| `cryptography/` | *(none yet)* | preamble fragment + reference PDF only |

`cryptography/` and `mathematical_methods/` both contain a file named
`differential_geometry.tex`. Despite the name it is **not a chapter** — it is a
byte-identical *preamble fragment* (no `\documentclass`, no `\begin{document}`)
that `AMMF.tex` `\input`s before `\begin{document}`. `cryptography/` has no main
file at all, so it currently does not build; creating one means writing an
`AMMF.tex`-style wrapper around that fragment.

`mathematical_methods/preview.*` are the artifacts of a partial-compile preview
run, not a separate document.

## The two preamble families

**Dense two-column family** (`multivariate_methods`, `math_self_study`) — 10pt,
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

**Boxed-theorem family** (`mathematical_methods`, `cryptography`) — `report`
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
