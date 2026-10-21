# [docs] Clarify segment scopes requirements in OpenAPI

| | |
|---|---|
| **Project** | [FeatBit](https://github.com/featbit/featbit) (open-source feature flag platform) |
| **Pull request** | [featbit/featbit#1017](https://github.com/featbit/featbit/pull/1017) |
| **Type** | `docs` |
| **Stack** | C#, ASP.NET Core, OpenAPI, FluentValidation, xUnit |
| **Status** | ✅ Merged |
| **Date** | 2026-10 |
| **Related issue** | [featbit/featbit#1015](https://github.com/featbit/featbit/issues/1015) |

## Problem

The public API reference showed `scopes` on the "create segment" request as optional and nullable, with no explanation of its format. The API actually rejects a missing or empty list with `scopes_is_invalid`, so anyone building a client from the spec got a request the server refused.

## How I found it

I was looking for C#/.NET issues in mid-sized projects and picked #1015. I claimed it on the issue first, and a maintainer confirmed the approach and clarified the scope rules (keys not IDs, one scope for environment-specific segments, one or more for shared ones, no wildcards)

## The fix

- Documented the `scopes` contract on `CreateSegment.Scopes` (XML docs plus a JSON example), following the maintainer's rules.
- Added a validator test for the "scopes omitted" case from the issue.
- Marked `scopes` as required and non-empty in the generated schema. My first version did this with a Swashbuckle schema filter so request validation stayed untouched, and I verified it against the real generated `swagger.json` (before/after)

## Review

While the PR was open, the project migrated from Swashbuckle to Microsoft.AspNetCore.OpenApi (#1006), which removed the filter's API. The maintainer merged `main` into my branch and replaced the filter with `[Required]` / `[MinLength(1)]` annotations, which the new generator reads directly. The docs and the test were merged as I wrote them.

## What I learned

- Check open PRs that touch the same area before coding; a large refactor (#1006) can change the right solution mid-review.
- In ASP.NET Core, data annotations on `[ApiController]` models trigger automatic model validation, which is a trade-off against custom validation responses.
- How to verify generated OpenAPI output without Docker, by fetching it from the in-memory test host.
