# nomad-gallery

`nomad-gallery` is the source repository for the NOMAD Gallery website: a MkDocs-based site for showcasing NOMAD apps, datasets, workflows, and community use cases.

This repository is only partly a standard NOMAD plugin repository. The Python package skeleton is still present, but the day-to-day maintenance work is mostly in the documentation content and the GitHub submission workflow.

## What lives where

The main repository areas are:

- `docs/index.md`
  The homepage. This includes the top featured carousel and the main gallery section.
- `docs/cards/`
  Regular gallery cards shown in the main gallery grid.
- `docs/special_cards/`
  Hand-picked cards used in the featured carousel at the top of the homepage.
- `docs/assets/`
  Local images and other static assets used by cards and pages.
- `.github/ISSUE_TEMPLATE/submission.yml`
  The public GitHub submission form.
- `.github/workflows/create-submission.yml`
  The automation that converts approved submissions into markdown card files and opens a PR.
- `main.py`
  MkDocs macros and card-rendering logic.
- `site/`
  Generated site output. This is build output, not the primary source of truth for content edits.

## How the normal submission flow works

Most new gallery entries should go through the GitHub submission process rather than being added manually.

### 1. A user submits through the GitHub issue form

The form is defined in `.github/ISSUE_TEMPLATE/submission.yml`.

It collects:

- submitter and affiliation
- title, description, research field, methodology, and technique
- image URL
- contributors and keywords
- NOMAD resource links
- publication, repository, funding, and media references
- optional metrics such as data size, active users, and downloads

### 2. A maintainer reviews the issue

When a submission is complete, add the label:

- `submission-approved`

### 3. GitHub Actions generates the card PR

The workflow in `.github/workflows/create-submission.yml`:

- parses the issue form
- normalizes the submitted fields
- writes a markdown card under `docs/cards/issue-<number>.md`
- opens a PR on a `submission/issue-<number>` branch

### 4. A maintainer updates the generated PR if needed

Typical follow-up edits include:

- uploading the image into the repository and replacing the temporary external image URL
- fixing wording, formatting, or metadata
- cleaning up references and contributors

### 5. The PR is merged

After checks pass, merge the PR. The card then becomes part of the gallery.

## Manual card editing

Manual edits are still useful for:

- fixing existing cards
- curating older entries
- removing outdated or placeholder cards
- maintaining the featured carousel

For regular gallery entries, edit files in `docs/cards/`.

For images stored in the repository, prefer a stable local asset path under `docs/assets/` or a subdirectory inside it.

## Featured carousel at the top of the homepage

The top carousel is curated manually. It is not automatically derived from `docs/cards/`.

The relevant files are:

- `docs/index.md`
- `docs/special_cards/*.md`
- `main.py` via `render_featured_rotator_card(...)`

### How to update the featured carousel

1. Create or update a markdown file in `docs/special_cards/`.
2. Open `docs/index.md` and find the `featured-rotator__slides-template` block.
3. Add or remove the corresponding `render_featured_rotator_card(...)` entry there.
4. In the same file, update the `featured-rotator__dots` block so the number of dots matches the number of slides.
5. Verify that the slide image renders correctly and that the “View in Gallery” button points to the intended main gallery card.

The two places you usually need to edit in `docs/index.md` are:

```html
<div class="featured-rotator__dots">
  <button class="featured-rotator__dot is-active" type="button" aria-label="Go to slide 1"></button>
  <button class="featured-rotator__dot" type="button" aria-label="Go to slide 2"></button>
</div>
```

and:

```jinja
<template class="featured-rotator__slides-template">
  <div class="featured-rotator__slide-source">
    {{ render_featured_rotator_card("special_cards/perovskite_database.md", index=1) }}
  </div>
</template>
```

To add another featured card, add another `featured-rotator__slide-source` block and a matching dot button.

### Important implementation details

#### The slide list is manual

The featured slides are declared explicitly in `docs/index.md` inside the `featured-rotator__slides-template` block.

For example:

```jinja
<div class="featured-rotator__slide-source">
  {{ render_featured_rotator_card("special_cards/perovskite_database.md", index=1) }}
</div>
```

Adding a file to `docs/special_cards/` alone does not make it appear in the carousel.

#### The dot count is also manual

The buttons in the `featured-rotator__dots` block are not generated automatically.

For example:

```html
<div class="featured-rotator__dots">
  <button class="featured-rotator__dot is-active" type="button" aria-label="Go to slide 1"></button>
  <button class="featured-rotator__dot" type="button" aria-label="Go to slide 2"></button>
</div>
```

If you add or remove a slide, update the dot buttons as well.

#### Special-card filenames should match the regular card slug

The carousel’s “View in Gallery” button derives its target from the `docs/special_cards/<name>.md` filename.

In practice, a special card should usually have a matching regular card such as:

- `docs/special_cards/perovskite_database.md`
- `docs/cards/perovskite_database.md`

If the filenames do not align, the button may not jump to the intended card in the main gallery.

#### Only part of the metadata is used by the top carousel card

The featured-card renderer mainly uses:

- `title`
- `research_field`
- `image_path`
- `image_name`

Other metadata can still be stored in the file for consistency, but it is not all shown in the compact carousel card itself.

## Local development

Create and activate a virtual environment:

```sh
git clone https://github.com/FAIRmat-NFDI/nomad-gallery.git
cd nomad-gallery
python3.11 -m venv .pyenv
. .pyenv/bin/activate
pip install --upgrade pip
pip install uv
```

Install the package in editable mode:

```sh
uv pip install -e '.[dev]' --index-url https://gitlab.mpcdf.mpg.de/api/v4/projects/2187/packages/pypi/simple
```

Serve the docs locally:

```sh
mkdocs serve
```

Run tests:

```sh
python -m pytest -sv tests
```

Run linting and formatting checks:

```sh
ruff check .
ruff format . --check
```

## Practical maintenance advice

- Treat `docs/` as the source of truth.
- Treat `site/` as generated output unless you intentionally need to update built artifacts.
- Prefer the GitHub submission flow for ordinary cards.
- Reserve `docs/special_cards/` for manually curated featured entries.
- When removing cards, also remove homepage references to them if they are featured.
- When changing submission metadata semantics, check the issue form, workflow, template, and renderer together.

## Main contributors

| Name | E-mail |
|------|--------|
| Joseph Rudzinski | [joseph.rudzinski@physik.hu-berlin.de](mailto:joseph.rudzinski@physik.hu-berlin.de) |
