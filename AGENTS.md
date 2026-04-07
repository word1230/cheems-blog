# Repository Guidelines

## Project Structure & Module Organization
This repository is a Hugo site using the FixIt theme. Write content in `content/`, grouped by section such as `content/posts/`, `content/java/`, and `content/系统/`. Shared defaults live in `archetypes/default.md`. Site-level overrides belong in `assets/css/_custom.scss`, `assets/js/custom.js`, and `layouts/`. Generated output is committed under `public/`; do not hand-edit it. The theme is tracked as a submodule at `themes/FixIt`, so initialize it before local development.

## Build, Test, and Development Commands
- `git submodule update --init --recursive`: fetch the FixIt theme before running Hugo.
- `hugo server -D`: start the local dev server and include draft content.
- `hugo --gc --minify`: produce a production build and refresh `public/`.
- `hugo new content/posts/<name>.md`: create a new post from the default archetype.

Use Hugo Extended. `hugo.toml` sets a minimum of `0.147.7`; the local environment currently has `0.159.1+extended`.

## Coding Style & Naming Conventions
Use two-space indentation in Hugo templates, SCSS, and JavaScript to match existing files in `layouts/` and `assets/`. Keep customizations in the small override surface rather than editing generated output or vendored theme files. Prefer descriptive lowercase filenames for content, and preserve the existing section layout. Front matter should stay in YAML/TOML-compatible key-value form with fields such as `title`, `date`, `slug`, `categories`, and `collections`.

## Testing Guidelines
There is no automated test suite in this repository. Validation is build-based: run `hugo --gc --minify` and confirm it completes cleanly, then spot-check the affected pages in `hugo server -D`. When editing templates, styles, or scripts, verify both desktop and mobile rendering and review the generated diff in `public/`.

## Commit & Pull Request Guidelines
Recent history uses short, imperative subjects such as `更新博客样式`; keep commit titles brief and specific. Avoid generic messages like `up` unless the change is truly trivial, and keep backup-only commits out of review branches. Pull requests should summarize the user-visible change, list touched areas such as `content/`, `assets/`, or `layouts/`, and include screenshots for visual changes. If `public/` changes, mention that the build output was regenerated intentionally.
