---
name: terraform
description: >
  Provides conventions for authoring and maintaining reusable Terraform
  modules across cloud providers (AWS, GCP, and others) — module directory
  layout, variable/output patterns, conditional resources, cross-variable
  validation, provider-specific quirks, and the validate + tflint workflow.
  Use when adding a variable, creating a resource, refactoring or
  scaffolding a module, or running validate/lint on a module. Triggers on:
  "add variable", "create resource", "new module", "terraform module",
  "validate", "tflint", "run lint", "gcp module", "aws module", or any
  request that touches a Terraform module.
metadata:
  tags: terraform, iac, aws, gcp, multi-cloud, hcl, validate, tflint
hooks:
  PostToolUse:
    - matcher: "Edit|Write|MultiEdit"
      hooks:
        - type: command
          timeout: 120
          statusMessage: "terraform fmt + validate"
          command: |
            f=$(jq -r '.tool_input.file_path // empty')
            case "$f" in *.tf|*.tfvars) ;; *) exit 0 ;; esac
            ctx() { jq -n --arg c "$1" '{hookSpecificOutput:{hookEventName:"PostToolUse",additionalContext:$c}}'; }
            block() { jq -n --arg r "$1" '{decision:"block",reason:$r}'; }
            if ! command -v terraform >/dev/null; then
              ctx "terraform is not installed, so fmt/validate did not run. Tell the user and list the commands to run: terraform fmt, terraform init -backend=false, terraform validate."; exit 0
            fi
            d=$(dirname "$f")
            if ! out=$(terraform fmt -no-color "$f" 2>&1); then block "terraform fmt failed on $f:
            $out"; exit 0; fi
            if out=$(terraform -chdir="$d" validate -no-color 2>&1); then ctx "terraform fmt + validate passed in $d."; exit 0; fi
            if grep -qiE 'terraform init|missing required provider|module not installed|not yet installed|inconsistent dependency lock' <<<"$out"; then
              ctx "terraform fmt ok. terraform validate needs init: run 'terraform init -backend=false' in $d, then 'terraform validate'."
            else
              block "terraform validate failed in $d:
            $out"
            fi
---

## When to use

Use this skill whenever you are authoring, updating, or validating a
reusable Terraform module — for AWS, GCP, or another provider — adding
variables, creating resources, refactoring a module's structure, or
running `terraform validate` / `tflint`.

## Core Principle

A module's public surface is its `variables.tf` and `outputs.tf` — treat
changes there as an API contract. Prefer expressing constraints declaratively
(`variable` validation blocks, `lifecycle { precondition {} }`) over relying
on convention or documentation alone, so a misuse fails at `validate`/`plan`
time instead of silently producing broken infrastructure.

## Common Workflows

**Scaffolding or navigating a module**: every module follows the same
directory shape (`main.tf`, `variables.tf`, `outputs.tf`, `versions.tf`,
`examples/`). See [rules/module-structure.md](rules/module-structure.md).

**Creating a resource**: Is it conditional? → `count = var.create_<thing> ?
1 : 0`, never `for_each` for a boolean toggle. Has an optional nested block?
→ `dynamic` block gated on a `for_each` ternary. Needs a check that spans
multiple variables? → `lifecycle { precondition {} }` on the resource, not a
`variable` validation block. These patterns are the same for any provider —
see [rules/conventions.md](rules/conventions.md). For how a specific
provider's resources behave differently (AWS vs GCP), see
[rules/provider-quirks.md](rules/provider-quirks.md).

**Adding a variable**: place it near the variable it logically follows, put
boolean toggles before their companion config-object variable, and put
cross-variable checks on the resource, not the variable block. See
[rules/adding-variables.md](rules/adding-variables.md).

**Validating a change**: once this skill is loaded, a `PostToolUse` hook
(in this file's frontmatter) runs `terraform fmt` on every edited `*.tf` /
`*.tfvars` file and `terraform validate` on its directory, feeding failures
back to Claude. It never runs `init` itself — if validate needs it, run
`terraform init -backend=false` once in the module directory. `tflint
--recursive` is still a manual step. See
[rules/validate-and-lint.md](rules/validate-and-lint.md).

**Something looks wrong but validate/lint pass**: check
[rules/gotchas.md](rules/gotchas.md) — several classes of error (provider
API constraints, cross-variable conditions) only surface at `plan` time,
not `validate` time.

**Working with a provider you haven't used in a module before (AWS vs GCP,
etc.)**: don't assume one provider's resource-schema behavior (e.g. AWS's
null-vs-empty-list handling) carries over to another — check
[rules/provider-quirks.md](rules/provider-quirks.md) and the provider's own
resource docs.

## How to use

Read individual rule files for detailed explanations and code examples:

- [rules/module-structure.md](rules/module-structure.md) - Module directory layout and the examples/ convention
- [rules/conventions.md](rules/conventions.md) - Version pins, conditional resources, dynamic blocks, preconditions, output patterns (provider-agnostic)
- [rules/provider-quirks.md](rules/provider-quirks.md) - Provider-specific resource schema quirks (AWS, GCP)
- [rules/adding-variables.md](rules/adding-variables.md) - Workflow for adding a new input variable
- [rules/validate-and-lint.md](rules/validate-and-lint.md) - terraform validate + tflint workflow
- [rules/gotchas.md](rules/gotchas.md) - Limits of validate/lint, tflint setup, version requirements
