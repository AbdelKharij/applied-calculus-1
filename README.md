# Applied Calculus 1 (MATH 1013 M) — Ximera edition

Ximera conversion of Aram Dermenjian's fill-in-the-blank lecture notes for Applied Calculus 1
(York University, Winter 2020). Source: `MATH1013M_student_notes.pdf`, OCR'd with Mathpix and then
corrected by hand.

**The course is published at <https://xerxes.ximera.org/AbdelKharij-applied-calculus-1>.**
Every push to `main` rebuilds and republishes it; see [Publishing](#publishing) below.

Built from the [ximeraNewProject](https://github.com/XimeraProject/ximeraNewProject) template.

## Layout

```
appliedCalculus1.tex   the xourse file: one \section per week, one \activity per section
xmPreamble.tex         shared preamble; ximera.cls loads it automatically for every document
xmPictures/            every figure from the original notes (mostly blank grids for sketching)
week00/ … week14/      one folder per week; one activity .tex per numbered section
.github/workflows/     publish.yml — builds and publishes on every push to main
xmScripts/, .vscode/, .devcontainer/, global.css, .gitignore   from the Ximera project template
```

Each activity is a standalone Ximera document, and does **not** input the preamble itself:

```latex
\documentclass{ximera}
\title{Limit laws}
\begin{document}
\begin{abstract} … Textbook §2.3. \end{abstract}
\maketitle
…
\end{document}
```

`ximera.cls` searches `./ ../ ../../ ../../../` for `xmPreamble.tex` and for a `xmPictures/`
folder, so both work from any depth without relative paths in the activities. This matters:
`xourse.cls` disables `\input` while inlining an activity, so an activity that inputs its own
preamble compiles standalone but silently loses those macros in the xourse build.

## How the fill-in blanks were converted

The original notes leave gaps for students to fill in during lecture. In this edition every gap is one of:

| In the original | Here |
|---|---|
| a blank with a definite mathematical answer | `\answer{…}` (Ximera checks it) |
| a blank word (even / odd, …) | `\wordChoice{\choice[correct]{…}\choice{…}}` |
| an example the lecturer invents live | a concrete example, marked `% AUTHOR CHOICE` in the source |
| open work space for a proof or sketch | `\begin{freeResponse}\end{freeResponse}` |
| admin details (office hours, exam room) | "(to be announced)" |

Search the source for `AUTHOR CHOICE` to find every example chosen during conversion, and for `TODO`
to find the handful of blanks whose intended content could not be determined.

Compiling an activity with `\documentclass[handout]{ximera}` prints the answers as blanks again, which
reproduces the original lecture handout.

## Publishing

`.github/workflows/publish.yml` runs on every push to `main` (and can be started by hand from the
Actions tab). It runs the standard Ximera pipeline in the project's Docker image —

```
xmlatex bake -j 5   # compile every activity and the xourse to HTML
xmlatex frost       # commit the built output and tag it
xmlatex name        # derive the course name
xmlatex serve -f    # upload to the Ximera server
```

— publishing to `https://xerxes.ximera.org/` under `AbdelKharij-applied-calculus-1`.

Two repository secrets drive it, both set already:

| Secret | What it is |
|---|---|
| `GPG_KEY` | base64 of an ASCII-armored private GPG key; identifies this repo to the Ximera server |
| `GPG_KEY_ID` | that key's 40-character fingerprint |

To rotate them, generate a new key and replace both secrets — nothing else refers to the old one:

```
gpg --batch --gen-key <params>
gh secret set GPG_KEY_ID --body "<fingerprint>"
gpg --armor --export-secret-keys <fingerprint> | base64 | tr -d '\n' | gh secret set GPG_KEY
```

Alternatively `xmlatex genKey` writes a `.xmKeyFile` that `xmlatex` picks up when the environment
variables are unset. `.gitignore` excludes `.xmKeyFile*`; keep it that way.

Note that `xerxes.ximera.org` is XimeraProject's public test server. Anything published there is
world-readable, and the operators make no durability promise — treat the repo, not the server, as
the source of truth.

## Compiling locally

The full toolchain ships as a Docker image, so nothing but Docker is needed:

```
./xmScripts/xmlatex bake -j 5
```

With a local TeX Live (2025 or newer, including the `ximera` package) you can also compile a single
activity straight to PDF. The build tools `cd` into the activity's own folder first, so do the same:

```
cd week03 && pdflatex limit-laws.tex
```

Compiling an activity with `\documentclass[handout]{ximera}` prints the answers as blanks again,
which reproduces the original lecture handout.

### Two things that bite in the HTML build

- **Font encoding.** `xmPreamble.tex` loads `fontenc`/`lmodern` and `newunicodechar` only when the
  engine is *not* LuaTeX. The HTML build runs `dvilualatex`, where forcing T1 with a Type1 font
  turns `§` into `ğ` and drops en-dashes; LuaTeX's native Unicode encoding handles them, and `ſ`
  (U+017F) — which the 8-bit engines need mapped to the `yfonts` Fraktur `\longs` — passes straight
  through to the HTML.
- **Missing figures.** The figures are JPEGs. A TeX4ht config that clears
  `\Configure{graphics*}{jpg}` (or no-ops `\Gin@setfile`) silently drops every one of them: the
  `image` environment still emits its `<div class="image-environment">`, just with nothing inside.
  If the HTML comes out with no `<img>` tags, that suppression is what to look for, not the
  graphics paths.

The `\answer` boxes need a Ximera server to become input boxes. Opened as plain static files the
built pages render them as literal red `\answer{...}`, because the macro sits inside math mode and
TeX4ht hands it to MathJax verbatim. That is expected, not a build fault.

## Notes

- The long s (ſ) in the remark on the origin of the integral sign is typeset via the `yfonts`
  Fraktur font, because no standard text font carries U+017F. `xmPreamble.tex` maps the Unicode
  character to `\longs` for the 8-bit engines.
- The author's own typos and a few mathematical slips in the original were left as they were. They are
  listed under "Errors in the original" in the `OCR_CHECK_REPORT.md` of the source repository.
- Original theorem/example numbers are preserved as labels, e.g. `\label{exercise:6-2}`.

## Corrections to the original's mathematics made in this edition

The OCR clean-up left the author's slips untouched; the answer-key check for this edition fixed the
ones that would otherwise make an answer box wrong. Each is marked `% CHECKED-FIX` in the source:

- `week11/antiderivatives.tex`: power rule condition `n ≠ 1` → `n ≠ -1`; derivative of arccot
  `-1/√(1+x²)` → `-1/(1+x²)`.
- `week09/hyperbolic-functions.tex`: `sinh(x+y) = sinh(x)cosh(x) + …` → `sinh(x)cosh(y) + …`.
- `week09/differentials.tex`: "we call dx and y differentials" → "dx and dy".
- `week08/linear-approximations-and-differentials.tex`: the quoted calculator value in Example 8.10
  now matches the function chosen for Example 8.9.
- `week03/continuity.tex` and `week12/definite-integral.tex`: word-choice distractors that were
  accidentally also true were replaced.

Still the author's call, left as in the original: the `sin(x)/ln(x)` limit in Week 10 is treated as
a 0/0 form (it is really 0/(-∞)); the two-car related-rates exercise in Week 8 has inconsistent data;
the missing `=` in `dy/dt ky`; the `xtoa` typo in the squeeze theorem; the Avogadro exponent; and
the spelling typos.

## Answer types to test once deployed

Ximera checks `\answer` by evaluating the expression numerically. About 60 of the 501 answer boxes
contain things a numeric checker cannot evaluate, kept because in handout mode they print as the
original blanks and the intent is clear from context:

- limit definitions and difference quotients in an unspecified `f`, e.g. `\answer{f'(g(x))\,g'(x)}`,
  the seven limit laws, the product/quotient/chain rules, `\answer{F(b)-F(a)}`;
- integral and sum notation, e.g. `\answer{\int_a^b f(x)\,dx}`, `\answer{a_n - a_0}`;
- set answers: `\answer{\mathbb{R}}` (10 places, Week 1–2), interval unions with `\cup` (Week 10);
- `\answer{\infty}` / `\answer{-\infty}` (Weeks 2 and 4);
- answers containing an arbitrary constant `+C` (Weeks 11 and 13) or a free parameter (`k`, `n`).

If any of these misbehave online, the quickest fix is to turn that box into a `\wordChoice`. Grep for
`\answer{\lim`, `\answer{\int`, `\answer{\mathbb`, `\answer{\infty`, `\answer{-\infty`, `\cup` to find them.
