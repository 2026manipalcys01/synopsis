# Mini-Project Synopsis — Manipal School of Information Sciences

A LaTeX recreation of the official MSIS (MAHE, Manipal) mini-project synopsis
template. It reproduces the cover page — the Manipal Academy header logo, the
orange project title, the *submitted to* block, the Reg. Number / Name / Branch
table, the date, and the school footer logo — plus a running footer, and a body
with the four required sections. It builds locally, in Docker, on Overleaf, and
on every push through GitHub Actions.

## What you get

- A faithful cover page: header logo, orange title, student table (dashed rules), date, footer logo
- A running footer (`Manipal School of Information Sciences` · page number) on every page
- Nine sections, one file each (abstract through expected outcomes), plus a gap-analysis and functional-requirements table
- Four diagrams written in Mermaid and embedded as vector graphics
- Times-like serif type (`mathptmx`) and `microtype` for clean output
- CI that compiles the PDF on every push and attaches it to tagged releases

## Quick start

Everything runs inside one Docker image (TeX Live + Mermaid CLI + `rsvg-convert`),
so the only thing you install is Docker. From the repo root:

```bash
make
```

The first run builds the image, renders the diagrams, and writes `build/CYS01-Synopsis.pdf`.
Later runs reuse the image and re-render only the diagrams whose source changed.

Live preview that rebuilds on every save:

```bash
make watch
```

Other targets: `make diagrams` (render diagrams only), `make image` (rebuild the
image), `make clean`, `make cleanall`. `docker compose up` is equivalent to
`make watch`.

### Overleaf

Overleaf compiles `src/main.tex`, but diagram rendering runs through the Docker
toolchain here, so build locally with `make` for the finished PDF.

## Make it yours

Open `src/main.tex` and edit the two fields near the top:

```latex
\newcommand{\projecttitle}{Title of the Project}
\newcommand{\projectdate}{DD/MM/YYYY}
```

Then fill in the student rows on the cover. Each row is `Reg. Number & Name & Branch`;
add or remove rows as needed:

```latex
\begin{tabular}{:p{3.8cm}:p{4.0cm}:p{3.4cm}:}
\hdashline
\centering\bfseries Reg. Number & \centering\bfseries Name & \multicolumn{1}{c:}{\bfseries Branch}\\ \hdashline
 220911001 & Jane Doe   & MSIS \\ \hdashline
 220911002 & John Smith & MSIS \\ \hdashline
\end{tabular}
```

Write the content of each section in its file under `src/sections/`.

## Sections

Each section lives in its own file under `src/sections/`, and `main.tex` reads them
in order:

```latex
\input{sections/01-abstract}
\input{sections/02-introduction}
\input{sections/03-literature-review}
\input{sections/04-gap-analysis}
\input{sections/05-objectives}
\input{sections/06-architecture}
\input{sections/07-functional-requirements}
\input{sections/08-methodology}
\input{sections/09-outcomes}
```

To add a section, write `src/sections/NN-name.tex` and add one
`\input{sections/NN-name}` line in the order you want it to appear.

## Diagrams

Each diagram is a Mermaid source in `diagrams/`. Rendering it produces an `.svg`
in `src/figures/` (the tracked, editable asset) and a build-only `.pdf` in
`build/diagrams/` that pdfLaTeX embeds, because pdfLaTeX cannot read `.svg`
directly. No diagram PDF is ever tracked. `make` renders anything whose source
changed, so you normally do not run this by hand, but you can:

```bash
make diagrams
```

Until a diagram is rendered, it shows a labelled placeholder and the document
still compiles. To add a diagram, drop `NAME.mmd` in `diagrams/` and reference it
from a section with `\diagramfig{NAME}{caption}{fig:label}`.

The render step (`scripts/render-diagrams.sh`) runs `mmdc` with
`diagrams/mermaid-config.json`, which keeps labels as plain SVG text so
`rsvg-convert` can turn each SVG into a vector PDF.

## Layout

```
.
├── Dockerfile            build image: TeX Live + Mermaid CLI + rsvg-convert
├── Makefile              make / make diagrams / make watch / make image
├── diagrams/             Mermaid sources
│   ├── *.mmd             one source per diagram
│   └── mermaid-config.json
├── scripts/
│   └── render-diagrams.sh  mmd -> src/figures/*.svg -> build/diagrams/*.pdf
├── src/
│   ├── main.tex          preamble, cover page, and the section order
│   ├── sections/         one file per section (01-abstract … 09-outcomes)
│   ├── figures/          logos (.png) and rendered diagrams (.svg)
│   └── references.bib    bibliography entries
├── build/                generated PDF, aux files, diagram PDFs (git-ignored)
├── .latexmkrc            build config: compiles src/, outputs to build/
├── docker-compose.yml    live-preview container (equals make watch)
└── .github/workflows/    CI that builds the PDF
```

The `.latexmkrc` compiles from inside `src/`, so `\input{sections/...}`,
`figures/`, and the embedded `../build/diagrams/*.pdf` all resolve against the
source, then it sends the output up to `build/` at the repo root.

## Continuous builds

Every push to `main` and every pull request builds the PDF and uploads it as an
artifact. Grab it from the run's summary page under Actions. To publish a
versioned PDF, tag a commit:

```bash
git tag v1.0
git push origin v1.0
```

The workflow builds the diagram PDFs from the committed SVGs with `rsvg-convert`,
then compiles, so it needs no Mermaid. Run `make diagrams` and commit the updated
SVGs after editing any `.mmd`.

## Requirements

- Docker and `make`. Nothing else is needed on the host.

The Mermaid CLI, `rsvg-convert`, and the full TeX Live toolchain all live inside
the image built from the `Dockerfile`.

## License

MIT. See [LICENSE](LICENSE).
