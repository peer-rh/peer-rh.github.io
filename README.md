# Academic GitHub Pages Site

This is a native Jekyll site for GitHub Pages.

## Local development

```sh
bundle install
bundle exec jekyll serve
```

The main content files are:

- `index.md` for the home page
- `publications.md` for the publications index
- `cv.md` for the CV page
- `_publications/*.md` for individual publications and project pages

Each publication supports front matter fields for `github`, `arxiv`, `paper`, `bibtex`, and `project`. If `project` is omitted, the project button links to the publication's local page.
