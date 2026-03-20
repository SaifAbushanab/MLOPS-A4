# Bug Report — GitHub Actions ML Pipeline (Assignment 4)

## Original (Buggy) YAML

```yaml
# .github/workflows/ml-pipeline.yml
name: ML Model CI

on:
push:
branches: main
pull_request:

jobs:
validate-and-test:
runs-on: ubuntu-latest
steps:
- name: Set up Python
uses: actions/setup-python@v5
with:
python-version: '3.10'

- name: Install Dependencies
run: pip install -r requirements.txt

- name: Linter Check

- name: Model Dry Test
run: |
python -c "import torch; print('Model environment ready!')"
```

---

## Bugs Identified & Fixes Applied

### Bug 1 — Missing Repository Checkout Step

| | Detail |
|---|--------|
| **Problem** | The workflow never checks out the repository code. Without a checkout step, files like `requirements.txt`, `train.py`, and `README.md` do not exist in the runner, causing every subsequent step that references them to fail. |
| **Root Cause** | The `actions/checkout` step was omitted from the original YAML. |
| **Fix** | Added `uses: actions/checkout@v4` as the **first** step in the job. |

```diff
     steps:
+      - name: Checkout Repository
+        uses: actions/checkout@v4
+
       - name: Set up Python
```

---

### Bug 2 — Invalid Indentation Under `on:` Trigger

| | Detail |
|---|--------|
| **Problem** | `push:`, `branches:`, and `pull_request:` are not properly indented under `on:`, making the YAML syntactically invalid. GitHub Actions will refuse to parse the file. |
| **Root Cause** | The nested keys were placed at the same level as `on:` instead of being indented beneath it. |
| **Fix** | Indented `push:` and its child `branches:` correctly under `on:`. Removed `pull_request:` since the assignment asks for push-only triggers. |

```diff
 on:
-push:
-branches: main
-pull_request:
+  push:
+    branches-ignore:
+      - main
```

---

### Bug 3 — `branches` Value Is Not a YAML List

| | Detail |
|---|--------|
| **Problem** | `branches: main` is a bare scalar. While some YAML parsers accept this, the GitHub Actions schema expects a **list** for the `branches` / `branches-ignore` key. |
| **Root Cause** | Missing list syntax (`- main` or `[main]`). |
| **Fix** | Changed to proper list format: `- main`. |

```diff
-    branches: main
+    branches-ignore:
+      - main
```

> **Note:** This fix is combined with the trigger modification required by the assignment (run on all branches **except** `main`).

---

### Bug 4 — `Linter Check` Step Has No Action

| | Detail |
|---|--------|
| **Problem** | The `Linter Check` step has only a `name:` key — it is missing both `run:` and `uses:`. GitHub Actions requires every step to have at least one of these, so the workflow will fail validation. |
| **Root Cause** | The linting command was never added. |
| **Fix** | Added a `run:` key that executes `flake8` with reporting flags. |

```diff
       - name: Linter Check
-
+        run: flake8 . --count --show-source --statistics
```

---

### Bug 5 — Minimal Model Dry Test (Observation)

| | Detail |
|---|--------|
| **Problem** | The dry test only runs `import torch; print(...)`. This verifies that PyTorch is installed but does not test any model logic. |
| **Assessment** | This is not a syntax or runtime bug — the step executes successfully. It is noted as a **quality observation** rather than a blocking issue. |
| **Action** | Kept as-is per the original assignment scope. |

---

## Additional Modifications (Per Assignment Requirements)

### Trigger Change

Changed the `on:` trigger so the pipeline runs on every `push` to **all branches except `main`**:

```yaml
on:
  push:
    branches-ignore:
      - main
```

### Artifact Upload Step

Added a final step to upload `README.md` as a GitHub artifact named `project-doc`:

```yaml
      - name: Upload Project Doc
        uses: actions/upload-artifact@v4
        with:
          name: project-doc
          path: README.md
```

---

## Fixed YAML (Complete)

```yaml
name: ML Model CI

on:
  push:
    branches-ignore:
      - main

jobs:
  validate-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'

      - name: Install Dependencies
        run: pip install -r requirements.txt

      - name: Linter Check
        run: flake8 . --count --show-source --statistics

      - name: Model Dry Test
        run: |
          python -c "import torch; print('Model environment ready!')"

      - name: Upload Project Doc
        uses: actions/upload-artifact@v4
        with:
          name: project-doc
          path: README.md
```
