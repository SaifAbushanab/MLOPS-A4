# ML Model CI — Assignment 4

This project demonstrates a GitHub Actions CI pipeline for an ML (PyTorch) project.


## Pipeline Steps

1. **Checkout** — clone the repository
2. **Setup Python 3.10** — install the Python runtime
3. **Linter Check** — run `flake8` for code quality
4. **Model Dry Test** — verify PyTorch imports correctly
5. **Upload Artifact** — upload `README.md` as `project-doc`
