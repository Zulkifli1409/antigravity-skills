---
name: readme-writing
description: Write, rewrite, review, or restructure README files for software projects, libraries, CLI tools, research code, analysis scripts, and repository directories, using merged guidance from several README and documentation guides. Use this skill whenever the user asks to make a README, bikin README, buat dokumentasi proyek, perbaiki atau review README, README.md template, dokumentasi repo GitHub, install and usage section, project description for a repo, or says things like my repo has no docs, help me document my code, make my project look professional, prepare my code for publication, supplementary code for my paper, or release my project as open source, even if they never say the word README or skill. Also use it for package-level READMEs in a monorepo, README quality checklists, and deciding what belongs in the README versus a wiki or docs folder.
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: Art of README by noffle (GitHub, n.d.); README Files for Code by ASAP Open Science Team (ASAP, 2024); How to write software documentation by Write the Docs (Write the Docs, n.d.); Write the Docs Documentation Release 1.0 by Eric Holscher and Troy Howard (2016), only sections 1.1-1.2 and 6.1 intro; About the repository README file (GitHub Docs, 2026); Google Style Guides - READMEs (Google, n.d.); Author Software README template (Cornell research code guidance, n.d.)
---

# README Writing

## 1. Operating instruction

Reply in the language the user writes in (Indonesian or English), go straight to the point, and deliver a ready-to-paste `README.md` (or a precise review with fixes) instead of advice about READMEs. Never invent commands, versions, licenses, or contacts. If a fact is unknown, insert a visible placeholder like `<TODO: ...>` and list at most three short questions at the end.

## 2. Choose the approach

First inspect what exists (repo files, package manifest, Makefile, LICENSE, example scripts, user description). Then pick one profile.

| Situation | Profile | Core idea |
|---|---|---|
| Reusable library, module, CLI, or tool others will adopt | A. Library/Tool | Reader funnel: broad to specific, let people decide fast whether to use it |
| Code for a single study, paper, or data analysis pipeline | B. Research code | Reproducibility: someone outside the team can run it and map scripts to outputs |
| Research software meant for wide reuse, with funding or citation needs | C. Research software (full) | B plus metadata: version, contacts, funding, related outputs |
| A package or directory inside a larger codebase | D. Package-level | Short summary, owners, status, how to use, links to real docs |
| User only wants a quick starter | A, minimal | Name, one-liner, example, install, license |

Also decide the audience before writing: **users** (want to use it, do not care how it works) and **developers** (want to contribute). Serve users first; give developers a separate, shorter section or a separate file.

Where to put the file:
- Name it exactly `README.md` (Markdown is the common default; reStructuredText works too).
- Put it at the repository root. GitHub also shows a README from `.github/` or `docs/`; if several exist, it picks `.github/` first, then root, then `docs/`.
- For package-level READMEs in large codebases, place them in the top-level directory of that package, not in its docs folder, and keep one in each top-level package directory that other teams consume.
- GitHub truncates rendered README content beyond roughly 500 KiB, so move long material elsewhere.
- Use relative links (for example `docs/CONTRIBUTING.md`) because absolute links break in clones. Keep link text on a single line or the link will not render.

## 3. Core concepts and why they matter

### 3.1 The README is the product front door
It is often the first and sometimes the only thing a visitor reads. It must answer: what is this, why would I want it, how do I start, where do I get help, who maintains it. Missing README is a red flag; a bloated one is not a sign of quality either.

### 3.2 Reader funnel (ordering by how fast a reader can say no)
Order sections so a reader can bail out as early as possible with minimal effort, and so each deeper section serves only readers who are already interested. Predictable structure saves cognitive effort because people scan many projects. The goal is honest evaluation, not selling: let the work state its promise briefly so people either adopt it or move on.

### 3.3 Brevity with a pointer to depth
Keep the README as short as possible but not shorter. The README should contain what is needed to start using and contributing; longer material belongs in a wiki, a docs folder, or a hosted docs site, linked from the README. Use FAQ sections sparingly: they go stale, collect unrelated items, are hard to search, and tempt you to patch gaps instead of writing real docs. Fix the docs or the code when the same question repeats.

### 3.4 Docs define the interface
Documentation is complete when someone can use the project without reading its source. That lets you change internals freely while the documented interface stays stable. Keep the public surface small early on, because every API change costs documentation work.

### 3.5 Why write it at all
Future you will forget the reasons behind decisions; people adopt only what they understand; contributors need guidance (and docs fixes are an easy first contribution); writing out the design often exposes design flaws.

### 3.6 Reproducibility for research code
Assume the reader has limited prior knowledge. State what data goes in, what the scripts do to it, exact steps, and what comes out. Mention which script produces which result. If data cannot be shared (for example protected health data), provide simulated dummy data so the code can still be verified.

### 3.7 Make it survive
The README outlives the hosting site and external images. Inline anything essential (short examples, key diagrams in text form) instead of relying on remote links. Link to specialized terms and related projects generously so readers can follow the background.

## 4. Workflow

1. **Collect facts.** Read manifests, scripts, license file, tests, and any existing docs. Note the language, entry points, inputs, outputs, supported platforms.
2. **Pick the profile** from section 2 and state the choice in one line.
3. **Write the one-liner first.** One sentence saying what it does and for whom, with no jargon the target reader would not know. If the name is vague, the one-liner must carry the meaning.
4. **Write a minimal working example** before anything else detailed. Prefer the most typical use case, copy-pasteable, with expected output shown. For CLIs show the command and its output; for small stateless function sets a REPL-style transcript works well.
5. **Write install steps.** Basic case in a couple of lines; link out for caveats. For source installs include clone, enter folder, build or install. Mention prerequisites (language runtime, OS, other tools) with versions.
6. **Fill API or detailed usage** only as far as needed. For each function or option state: purpose, parameters, which are optional and their defaults, types when not obvious, return values, errors or caveats. For an options object, list every accepted key. If a function needs a long explanation, consider that it may be too complex.
7. **Add support and contribution info.** Where to report bugs (issue tracker), where to ask questions, how to contribute (or link to CONTRIBUTING).
8. **State license** (one standard license; put a non-permissive license near the top to avoid surprises) and keep a LICENSE file in the repo. Add citation info for academic or research code.
9. **Run a reality check.** Every command must exist in the repo or be a standard tool. If you cannot verify, mark it `<TODO: verify>`. Where possible suggest the user tries a fresh machine or clean environment, since that exposes missing dependencies.
10. **Review with the checklist** (section 7) and fix before delivering.

Rules for specific parts:
- **Dependencies:** list versions. If only one version was tested, say that no other versions have been tested. Where possible export the dependency list (package manager freeze, session info, or a container definition) and include it in the repo.
- **Resources:** for heavy workloads mention CPU and memory needs so people can pick the right machine.
- **External data:** say where to download it and which folder it must be placed in.
- **Folder structure:** a short annotated tree of folders and key files, especially for research code.
- **Limitations:** state tested scale and known caveats plainly (for example the largest input size that was tried).
- **Badges:** use few. Each badge must give a typical reader real value; build status is better routed to maintainers by notification. Badge images depend on remote hosting and vanish offline. (+)

## 5. Examples (original, short)

### 5.1 Profile A input and output

Input from user: "bikin README untuk library Python kecil bernama `tinyslug` yang mengubah judul jadi URL slug."

Output (excerpt):

````markdown
# tinyslug

Turn any title into a clean, URL-safe slug, with Unicode handled for you.

```python
from tinyslug import slugify

slugify("Halo Dunia! Ini Tes")   # -> "halo-dunia-ini-tes"
```

## Install

```bash
pip install tinyslug
```

Requires Python 3.9+. <TODO: confirm minimum version>

## Usage

`slugify(text, sep="-", max_len=None)`

- `text` (str): input title.
- `sep` (str, optional): word separator, default `-`.
- `max_len` (int, optional): cut result to this length, default no limit.

Returns a `str`. Raises `TypeError` if `text` is not a string.

## Contribute and support

- Issues: <TODO: issue tracker link>
- Questions: open a discussion or issue.

## License

MIT, see [LICENSE](LICENSE).
````

### 5.2 Profile B excerpt

````markdown
# Gait Study Analysis Scripts

Scripts that clean raw sensor files and produce Figures 2-4 of the paper <TODO: link>.

## Run the example

```bash
bash run_example.sh data/dummy/   # writes results to outputs/
```

Expected: three PNG files in `outputs/` and a summary CSV.

## Requirements
Ubuntu 22.04, R 4.3 (no other versions tested), 8 GB RAM. Package list in `renv.lock`.

## Folder structure
- `scripts/01_clean.R` cleans raw files
- `scripts/02_figures.R` makes Figures 2-4
- `data/dummy/` simulated data (real data cannot be shared)

## Limitations
Tested on files up to <TODO: size>. Larger inputs are untested.
````

## 6. Output format

Deliver in this order:

1. One line naming the chosen profile and audience.
2. The full README inside a single fenced Markdown block (use a four-backtick fence when the README itself contains code fences).
3. A short "TODO / please confirm" list, maximum three items, only for facts you could not verify.
4. If reviewing an existing README: a findings list ordered by impact (what is missing, what is misleading, what should move to docs), followed by the corrected README or the exact edits.

### Template A (library or tool)

````markdown
# <Project name>

<One-sentence description.>

<Optional: 1-3 meaningful badges.>

## Background   <!-- only if concepts or motivation are not obvious -->

## Usage
<Minimal runnable example + expected output>

## Install
<Prerequisites + install command>

## API / Options
<Per function or flag: params, defaults, types, returns, caveats>

## Support
<Where to ask, issue tracker>

## Contribute
<How to contribute or link to CONTRIBUTING.md>

## License
<License name + LICENSE link>
````

Variation: if the license is restrictive, move License right under the one-liner. Some guides place Install right after the first example; either order is fine if used consistently.

### Template B (research code)

````markdown
# <Project title>

## Overview
<Purpose, related study, data types, supported file formats, outputs>

## Working example
<Commands with parameters, example outputs, dummy data note>

## System requirements and dependencies
<OS, CPU/RAM, software with versions, data location>

## Installation
<Dependency install, clone and setup commands>

## Folder structure
<Annotated tree>

## Limitations

## Citation
<How to cite code; link to paper or preprint>

## License

## Acknowledgements
<Contributors; funding statement if required by the funder>

## Collaboration (only if others will reuse the code)
<Contributing guide, code of conduct, issue templates, change log>
````

### Template C additions (research software)
Add near the top: version (semantic versioning), short description, date of creation or release, project size and organization (source, binaries, tests, configs, build scripts, docs). Add at the bottom: at least two contacts (name, role, ORCID, institution, email), funding with grant numbers, publications using the software, other locations where the project is hosted, related datasets or scripts. Only fill what the user provides.

### Template D (package-level)
Must contain or link: what the package is and what it is for; points of contact; status (for example deprecated or not for general release); how to use it (sample code or runnable test or run commands); links to full documentation.

## 7. Review checklist

- [ ] File is named `README.md` and sits in the correct location.
- [ ] First screen tells what it is, for whom, and why it exists, without jargon.
- [ ] Name is self-explanatory, or the one-liner compensates.
- [ ] A minimal example is present, is copy-pasteable, and shows expected output.
- [ ] Install steps cover prerequisites, versions, and the basic command.
- [ ] Section order follows the funnel: broad and decision-critical first, details later.
- [ ] API or options document optional params, defaults, types, return values, caveats.
- [ ] Reader can use the project without reading source code.
- [ ] Support channel and issue tracker are stated.
- [ ] Contribution path is stated (or intentionally absent for a closed project).
- [ ] License is named and a LICENSE file exists; restrictive licenses are visible early.
- [ ] Research code: data inputs, script-to-output mapping, dummy data, folder tree, limitations, citation.
- [ ] Dependencies have versions; untested versions are admitted honestly.
- [ ] Every command and path was verified or flagged `<TODO>`; nothing is invented.
- [ ] Relative links are used for repo files; link text is on one line.
- [ ] Essential information is inline, not only behind external links or remote images.
- [ ] Badges are few and each one earns its place.
- [ ] FAQ is absent, or small and current.
- [ ] Length is proportionate; deep material is moved to a wiki or docs folder and linked.
- [ ] Alt text for images and meaningful link text (not "click here"). (+)

## 8. Outdated parts of the sources and modern alternatives

- Write the Docs (2016): the Freenode IRC channel, Google Groups mailing list, `easy_install` or `sudo` install instructions, and the Sphinx theme list are dated. Use current community channels (for example a Discussions tab, Discord, or Matrix), `pip` inside a virtual environment, and check current theme or tool options. (+)
- Write the Docs template shows a BSD placeholder license: choose a license deliberately and use a license picker or SPDX identifier. (+)
- Art of README is written around Node and npm: keep the principles, but adapt the examples (package registry name, install command, REPL idea) to the user's ecosystem.
- Art of README footnote about capital letters sorting first no longer holds on most systems; the all-caps name is now just a visibility habit.
- Install examples such as a bare `make` assume Unix tooling: add Windows notes when relevant. (+)
- Additional modern items (not in the sources): add `CITATION.cff` for citable code (+); add `SECURITY.md` and issue or pull request templates in `.github/` (+); pin dependencies with a lockfile or container or devcontainer for reproducibility (+); run a CI job that executes the README example so it cannot rot (+); use a Mermaid diagram for architecture since GitHub renders it (+); archive releases with a DOI service for papers (+).
- Parts of Write the Docs not used here (company documentation culture, Sphinx setup, themes, presentation resources) are outside README scope and were not turned into rules.

## Credits

- Source material: Art of README by noffle (GitHub, n.d.); README Files for Code by ASAP Open Science Team (Aligning Science Across Parkinson's, 2024); How to write software documentation by Write the Docs (n.d.); Write the Docs Documentation Release 1.0 by Eric Holscher and Troy Howard (2016), sections 1.1-1.2 and the 6.1 intro; About the repository README file (GitHub Docs, 2026); Google Style Guides, READMEs section (Google, n.d.); Author Software README template (Cornell research code and software guidance, n.d., publisher inferred from the link inside the file). This skill is an original summary and application of the sources' ideas, not a reproduction of their text.
- Skill author: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
