---
name: gotchas
description: Limits of terraform validate/tflint, tflint setup, and version requirements
metadata:
  tags: terraform, tflint, gotchas, versions
---

# Gotchas

- **`tflint --init` is a one-time setup per machine** to download plugins.
  "All plugins are already installed" on a subsequent run is expected, not
  an error.
- **Scope `tflint --recursive` to the module you're changing.** Running it
  from the repo root lints every module in the collection at once — useful
  for a full audit, but too noisy to use as your everyday check while
  editing one module.
- **`terraform validate` does not check the provider's API constraints or
  cross-variable conditions.** It only verifies HCL syntax and type
  correctness. A `lifecycle { precondition {} }`, or an actual provider
  rejection (e.g. an invalid AWS instance-type/region combination, or a GCP
  quota/zone mismatch), only surfaces at `terraform plan` (or `apply`) time
  against real credentials. See [provider-quirks.md](provider-quirks.md)
  for known per-provider cases.
- **An `examples/basic-usage/main.tf` that references
  `terraform_required_version` needs its own `versions.tf`.** Without one,
  `tflint` warns because it can't resolve the required-version constraint
  for that example in isolation — add a `versions.tf` matching the module's
  to fix it.
- **`optional()` in a variable's object type needs Terraform 1.3+.** A
  parse error mentioning `optional` usually means `versions.tf`'s
  `required_version` hasn't been bumped to `>= 1.3` yet — see
  [conventions.md](conventions.md#version-pins).
