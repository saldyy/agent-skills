---
name: conventions
description: Version pins, conditional resources, dynamic blocks, preconditions, and output patterns for Terraform modules
metadata:
  tags: terraform, count, for_each, dynamic-blocks, lifecycle, preconditions, multi-cloud
---

# Module Conventions

These patterns are Terraform-core constructs (`count`, `for_each`, `dynamic`,
`lifecycle`) — they work identically no matter which provider's resources the
module wraps. Provider-specific behavior (AWS vs GCP vs others) lives in
[provider-quirks.md](provider-quirks.md), not here.

## Version pins

- `required_version = ">= 1.3"` in every module's `versions.tf` —
  `optional()` in object-typed variables needs Terraform 1.3+, and
  `lifecycle { precondition {} }` needs 1.2+. Pinning to 1.3 covers both,
  regardless of which provider(s) the module uses.
- `required_providers`: pin every provider the module declares to a version
  range appropriate for the resources it uses — e.g. `aws >= 5.0` for AWS,
  `google >= 5.0` for GCP. Document inline why if a narrower range is
  required (a specific resource/argument only available past some version).

## Conditional resources

Use `count` for a simple boolean toggle, never `for_each` — this applies to
any provider's resource type:

```hcl
resource "aws_instance" "this" {
  count = var.create_instance ? 1 : 0
  # ...
}
```

```hcl
resource "google_compute_instance" "this" {
  count = var.create_instance ? 1 : 0
  # ...
}
```

`for_each` is for keyed collections (multiple instances keyed by name/map) —
reach for it only when there's genuinely more than one possible instance,
not to express on/off.

## Dynamic blocks

- **Optional single nested block** — gate it on a one-or-zero-element list:

  ```hcl
  dynamic "ebs_block_device" {
    for_each = var.extra_volume != null ? [var.extra_volume] : []
    content {
      device_name = ebs_block_device.value.device_name
      volume_size = ebs_block_device.value.volume_size
    }
  }
  ```

- **Repeated nested block** — iterate the list directly:

  ```hcl
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port = ingress.value.from_port
      to_port   = ingress.value.to_port
    }
  }
  ```

  The same pattern applies to any provider's repeated nested block, e.g. a
  GCP `dynamic "node_config"` block iterating `var.node_pools`.

## Cross-variable validation

`variable { validation {} }` blocks can only see the variable they're
attached to. When a constraint spans multiple variables, express it as a
`lifecycle { precondition {} }` on the resource that depends on both —
regardless of provider:

```hcl
resource "aws_db_instance" "this" {
  count = var.create_db ? 1 : 0

  lifecycle {
    precondition {
      condition     = !var.multi_az || var.create_db
      error_message = "multi_az requires create_db to be true."
    }
  }
}
```

Note this only fires when the resource itself is being created (`count = 1`)
— it won't catch a bad combination if the toggle is off.

## Output patterns

Mirror the resource's conditional creation in its output:

```hcl
output "instance_id" {
  value = var.create_instance ? aws_instance.this[0].id : null
}
```

For GCP resources, the equivalent output is often `self_link` rather than an
AWS-style `arn`/`id`:

```hcl
output "instance_self_link" {
  value = var.create_instance ? google_compute_instance.this[0].self_link : null
}
```
