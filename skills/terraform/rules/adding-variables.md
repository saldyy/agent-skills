---
name: adding-variables
description: Workflow for adding a new input variable to a Terraform module
metadata:
  tags: terraform, variables, workflow
---

# Adding a Variable

1. Add the new `variable` block to `variables.tf`, placed after the variable
   it logically follows (group related inputs together rather than
   appending alphabetically or at the end of the file).
2. If it's a boolean toggle (`create_<thing>`), place it immediately before
   the config-object variable it gates — a reader should see the toggle and
   its payload together.
3. If the variable interacts with another variable in a way that can't be
   expressed in its own `validation` block, add the check as a
   `lifecycle { precondition {} }` on the relevant resource instead — see
   [conventions.md](conventions.md#cross-variable-validation). Don't try to
   force a cross-variable check into a single-variable `validation` block.
4. Regenerate `README.md` (`terraform-docs`) so the documented variable
   table matches `variables.tf`.
5. If the module has an `examples/basic-usage/main.tf`, consider whether the
   example should exercise the new variable — a variable that's never
   demonstrated is easy to leave broken.
