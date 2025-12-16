# Documentation

This directory contains the Sphinx documentation source for Conflux S3 Utils.

## Building Locally

To build the documentation locally:

1. Install the documentation dependencies using uv:
   ```bash
   uv sync --group docs
   ```

2. Build the HTML documentation:
   ```bash
   cd docs
   make html
   ```

3. Open the documentation in your browser:
   ```bash
   open build/html/index.html
   ```

## Documentation Structure

- `source/` - Documentation source files (reStructuredText)
  - `conf.py` - Sphinx configuration
  - `index.rst` - Main documentation page
  - `installation.rst` - Installation instructions
  - `quickstart.rst` - Quick start guide
  - `api.rst` - API reference (auto-generated from docstrings)
  - `examples.rst` - Usage examples
  - `_static/` - Static files (CSS, images, etc.)
  - `_templates/` - Custom Sphinx templates

- `build/` - Built documentation (generated, not in git)
- `Makefile` - Build commands for documentation

Documentation dependencies are managed in `pyproject.toml` under the `docs` dependency group.

## Continuous Deployment

Documentation is automatically built and deployed to GitHub Pages when changes are pushed to the `main` branch. The workflow is defined in `.github/workflows/docs.yaml`.

## Writing Documentation

The documentation uses reStructuredText (RST) format. Some helpful resources:

- [reStructuredText Primer](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html)
- [Sphinx Documentation](https://www.sphinx-doc.org/)
- [Read the Docs Theme](https://sphinx-rtd-theme.readthedocs.io/)

## Updating API Documentation

API documentation is automatically generated from docstrings in the source code. To update:

1. Update docstrings in the source code (use Google or NumPy style)
2. Rebuild the documentation with `make html`

The `sphinx.ext.autodoc` and `sphinx.ext.napoleon` extensions handle the conversion from docstrings to documentation.
