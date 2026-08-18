# College Report Template

A LaTeX template for college reports. Fill in your details, write your content, and get a clean PDF with a cover page, table of contents, math, code listings, tables, figures, and a numbered reference list. It builds locally, in Docker, on Overleaf, and on every push through GitHub Actions.

## What you get

- A cover page driven by a few `\newcommand` lines, so you never touch the layout
- Table of contents, running header, and page numbers already wired up
- Math via `amsmath`, code via `listings`, tables via `booktabs`, diagrams via TikZ
- A bibliography through `biblatex` and Biber, with IEEE numbered citations
- One-and-a-half line spacing and `microtype` for readable output
- CI that compiles the PDF on every push and attaches it to tagged releases

## Quick start

Pick whichever path matches your setup. All three write the PDF to `build/main.pdf`.

### Docker with live preview

Nothing to install except Docker. This watches your files and rebuilds on every save.

```bash
docker compose up
```

Edit `src/main.tex`, save, and `build/main.pdf` refreshes. Press `Ctrl-C` to stop.

### Docker, one build

```bash
docker run --rm -v "$PWD":/work -w /work texlive/texlive:latest latexmk
```

### Local TeX Live

You need TeX Live (the full install is easiest) with `latexmk` and `biber` on your path. Then from the repo root:

```bash
latexmk
```

### Overleaf

Upload the repo, open the menu, and set the main document to `src/main.tex`. Overleaf runs Biber for you.

## Make it yours

Open `src/main.tex` and edit the block under `Fill in your details here`:

```latex
\newcommand{\reporttitle}{Report Title}
\newcommand{\reportsubtitle}{A short, descriptive subtitle}
\newcommand{\coursename}{Course Name}
\newcommand{\coursecode}{CS0000}
\newcommand{\professor}{Prof. Name}
\newcommand{\studentname}{Your Name}
\newcommand{\studentid}{Roll / ID Number}
\newcommand{\institution}{Your Institution}
\newcommand{\department}{Department Name}
```

That is the only place you touch metadata. The cover page and headers read from it.

To add a reference, drop a BibTeX entry into `src/references.bib` and cite it with `\cite{key}`. Uncited entries stay out of the printed list, so keep as many as you want on hand.

For a real image, save it to `src/figures/` and include it:

```latex
\includegraphics[width=0.6\linewidth]{name}
```

## Sections

Each part of the report lives in its own file under `src/sections/`. The body of `main.tex` reads:

```latex
\input{sections/introduction}
\input{sections/method}
\input{sections/results}
\input{sections/conclusion}
```

To add a section, write `src/sections/analysis.tex`, start it with a `\section{...}` heading, and add one `\input{sections/analysis}` line in the order you want it to appear. Reorder or drop a part by moving or deleting its `\input` line. No page-break surprises, since `\input` drops the content straight in. Cross-references and citations work across files, and the live-preview build rebuilds whenever any section changes.

## Front matter and page numbers

The cover, abstract, and the three lists print with roman numerals (i, ii, iii), then the body restarts at arabic 1. That is the convention most reports follow, and switching the numbering style is also what keeps the internal PDF links unique, so you never see the "destination with the same identifier" warning.

Three lists sit between the contents and the body, each shown as its own entry inside the table of contents:

- List of Figures
- List of Tables
- Listings (the code blocks)

Turn any of them off with a flag near the top of `main.tex`. All four default to on:

```latex
\showtocfalse
\showfiguresfalse
\showtablesfalse
\showlistingsfalse
```

Each list carries a `\phantomsection` before its `\addcontentsline`, so the contents entry links to the right page instead of the section above it.

The template is single-sided, so a plain `\clearpage` starts each front-matter page and no blank numbered filler pages show up. If you switch the class to `twoside`, add the `emptypage` package to strip headers and numbers off the blank pages that `\cleardoublepage` inserts.

## Layout

```
.
├── src/
│   ├── main.tex          preamble, cover, and the section order
│   ├── sections/         one file per section
│   │   ├── introduction.tex
│   │   ├── method.tex
│   │   ├── results.tex
│   │   └── conclusion.tex
│   ├── references.bib    bibliography entries
│   └── figures/          images go here
├── build/                generated PDF and aux files (git-ignored)
├── .latexmkrc            build config: compiles src/, outputs to build/
├── docker-compose.yml    live-preview container
└── .github/workflows/    CI that builds the PDF
```

The `.latexmkrc` compiles from inside `src/`, so `\input{sections/...}`, `references.bib`, and `figures/` all resolve against the source, then it sends the output up to `build/` at the repo root. That keeps your working tree clean.

## Continuous builds

Every push to `main` and every pull request builds the PDF and uploads it as an artifact named `report-pdf`. Grab it from the run's summary page under Actions.

To publish a versioned PDF, tag a commit:

```bash
git tag v1.0
git push origin v1.0
```

The workflow attaches `build/main.pdf` to a GitHub Release for that tag.

## Requirements

One of:

- Docker, for the container paths above
- A local TeX Live install with `latexmk` and `biber`
- An Overleaf account

## License

MIT. See [LICENSE](LICENSE).
