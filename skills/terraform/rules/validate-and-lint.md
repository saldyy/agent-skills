---
name: validate-and-lint
description: terraform validate + tflint workflow to run after every module change
metadata:
  tags: terraform, tflint, validate, workflow
---

# Validate + Lint

Run after every change, from inside the module directory:

```bash
cd modules/<name>

# First-time, or after a required_providers version change:
terraform init -backend=false

# Validate HCL syntax and type correctness:
terraform validate

# Lint (style rules, unused declarations, missing required_version, etc.):
tflint --recursive
```

`tflint --recursive`, run from the module root, covers both the module and
its `examples/` subdirectory in one pass. Exit code `0` means clean; `2`
means there are warnings or errors to address.

`terraform init -backend=false` skips backend configuration, which isn't
needed just to validate/lint a module in isolation — only re-run it when the
provider version constraints change.

For the boundaries of what `validate` and `tflint` actually catch, see
[gotchas.md](gotchas.md).
