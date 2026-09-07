---
name: provider-quirks
description: Provider-specific resource schema quirks for AWS and GCP Terraform modules — null vs empty values, labels vs tags, project scoping
metadata:
  tags: terraform, aws, gcp, provider-quirks
---

# Provider-Specific Quirks

The patterns in [conventions.md](conventions.md) (`count`, `dynamic`,
`lifecycle` preconditions) are Terraform-core and identical across
providers. What differs by provider is what each provider's resource
schema will and won't accept. When a module wraps a new provider, check its
resource docs for the equivalent of the quirks below rather than assuming
AWS behavior carries over.

## AWS

`cidr_blocks` / `security_groups` in inline `aws_security_group` rules must
be `null`, not `[]`, when there's nothing to set — AWS rejects an empty list
for these arguments:

```hcl
ingress {
  cidr_blocks = length(var.allowed_cidrs) > 0 ? var.allowed_cidrs : null
}
```

Tags are a free-form `map(string)` (`var.tags`) — key/value casing and
characters are largely unconstrained, and most modules merge a module-level
default tag map with caller-supplied tags.

## GCP

- **`labels` instead of `tags`** — GCP's equivalent of AWS tags is
  `labels`, a `map(string)`, but keys and values are constrained: lowercase
  letters, digits, underscores, and dashes only, starting with a lowercase
  letter, max 63 characters. Uppercase or a leading digit in a label value
  will fail at `apply` time, not `validate` time — validate this in a
  `variable { validation {} }` block if the module accepts caller-supplied
  labels.
- **`project` is usually explicit** — unlike AWS, where the account/region
  come from the provider block or environment, GCP modules commonly expose
  a `project_id` (or `project`) variable and pass it explicitly to
  resources, since a single Terraform run often spans multiple GCP
  projects.
- **Outputs favor `self_link` over an ARN-style identifier** — most
  `google_*` resources expose `self_link` (and often `id`), not an
  AWS-style ARN. Mirror conditional creation the same way as AWS:
  `var.create_x ? google_x.this[0].self_link : null`.

## Adding a new provider's quirks here

When a module starts using a provider not covered above, add a section
following the same shape: the field(s) that behave unexpectedly, whether
the failure shows up at `validate`/`plan`/`apply` time, and the working
pattern to use instead.
