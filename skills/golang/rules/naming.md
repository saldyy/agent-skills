---
name: naming
description: Identifier naming decision flow for Go
metadata:
  tags: naming, mixedcaps, receivers, initialisms
---

# Naming in Go

Go names tend to be shorter than in other languages: a good name doesn't repeat what's already clear from context, and takes the surrounding scope into account rather than trying to be self-sufficient in isolation.

## Decision Flow

```
What are you naming?
├─ Package       → short, lowercase, singular noun; no underscores, no mixedCaps
├─ Interface     → method name + "-er" when single-method (Reader, Writer)
├─ Receiver      → 1-2 letter abbreviation of the type; same abbreviation on every method
├─ Constant      → MixedCaps, named by role not value; never ALL_CAPS
├─ Exported func → verb/verb-phrase in MixedCaps; no `Get` prefix on simple accessors
├─ Variable      → length proportional to scope distance
│                   ├─ tiny scope (a handful of lines) → single letter (i, n, r)
│                   ├─ medium scope                    → short word (count, buf)
│                   └─ package-level / wide scope       → descriptive (userAccountCount)
└─ Anything      → does it repeat the package/receiver/type context? If so, shorten it.
```

## MixedCaps, Always

All Go identifiers use MixedCaps (`camelCase`/`PascalCase`), not underscores. The only exceptions: test function names (`TestFoo_InvalidInput`), generated code, and cgo/OS interop identifiers.

## Package Names

Lowercase, no underscores, short and specific. Avoid grab-bag names like `util`, `common`, `helper` — a package called `util` invites unrelated code to accumulate in it. Prefer names that say what's in the package: `stringutil`, `httpauth`, `configloader`.

```go
// Good: user, oauth2, tabwriter
// Bad:  user_service, UserService, count (shadows a common local var)
```

## Interface Names

Name a single-method interface after its method plus `-er`: `Reader`, `Writer`, `Formatter`. When the method matches a canonical stdlib signature (`Read`, `Write`, `Close`, `String`), keep that exact signature so the type composes with the rest of the ecosystem (`io.Reader`, `fmt.Stringer`, etc.).

## Receiver Names

Always a short (1-2 letter) abbreviation of the type, and always the *same* abbreviation across every method on that type — mixing `c` and `cl` for the same receiver is a review flag:

```go
func (c *Client) Connect() { ... }
func (c *Client) Send()    { ... }
```

Never `this` or `self` — that's not Go idiom.

## Constants

MixedCaps, named for what the constant represents, not its value:

```go
const MaxRetries = 3
const defaultTimeout = 30 * time.Second
```

`Three` or `Port8080` as a constant name is a smell — if the value ever changes, the name becomes a lie.

## Initialisms

Keep initialisms (`URL`, `ID`, `HTTP`, `API`) at a single consistent case throughout — either fully uppercase or fully lowercase, never title-cased:

```go
// Good: HTTPClient, userID, ParseURL()
// Bad:  HttpClient, orderId, ParseUrl()
```

## Function and Method Names

No `Get` prefix on a plain accessor — the field name (capitalized) is the getter name: `Owner()` not `GetOwner()`. Use a verb like `Compute` or `Fetch` when the call actually does non-trivial work, so callers can tell "cheap field read" apart from "this might be slow." When two functions differ only by argument/return type, put the type at the end: `ParseInt()`, `ParseInt64()`.

## Variable Names

- Scope-proportional length: `i`, `v`, `n` are fine in a five-line loop; something like `pendingOrders` earns its length at function or package scope.
- Familiar single-letter conventions carry meaning on their own: `i` for an index, `r`/`w` for reader/writer.
- Don't encode the type in the name: `users` not `userSlice`, `name` not `nameString`.
- Prefix unexported package-level vars/consts with `_` to avoid accidental shadowing by a local of the same short name.

## Avoiding Repetition

A name shouldn't repeat information the caller already has from context:

- Package + symbol: `widget.New()`, not `widget.NewWidget()`.
- Receiver + method: `p.Name()`, not `p.ProjectName()` on a `Project` receiver `p`.
- Package + type: inside package `sqldb`, `Connection`, not `DBConnection`.

## Don't Shadow Predeclared Identifiers

Never use Go's built-in identifiers (`error`, `string`, `len`, `cap`, `append`, `copy`, `new`, `make`, ...) as a variable, parameter, or type name — they still compile, but they silently shadow the builtin for the rest of that scope and confuse anyone reading the function.
