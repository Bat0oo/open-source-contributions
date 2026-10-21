# [fix] Set default tenant for activerecord-tenanted models

| | |
|---|---|
| **Project** | [annotaterb](https://github.com/drwl/annotaterb) (Ruby gem that annotates Rails models with their schema) |
| **Pull request** | [drwl/annotaterb#413](https://github.com/drwl/annotaterb/pull/413) |
| **Type** | `fix` |
| **Stack** | Ruby, Rails 8, ActiveRecord, RSpec |
| **Status** | ✅ Merged |
| **Date** | 2026-10 |
| **Related issue** | [drwl/annotaterb#276](https://github.com/drwl/annotaterb/issues/276) |

## Problem

Running annotaterb in a Rails app that uses Basecamp's activerecord-tenanted gem crashed instead of annotating the models. The issue had been open for about a year.

## How I found it

At work we use an older annotation gem that's no longer maintained, so I was looking for a better alternative and found annotaterb. My first pick ([#402](https://github.com/drwl/annotaterb/issues/402)) was fixed by someone else before I opened a PR, so for #276 I claimed the issue before writing code.

## The fix

When annotaterb boots the app, no tenant is selected, so Rails' `reset_column_information` hits activerecord-tenanted's untenanted connection pool, which has no `active_connection`. The fix makes annotaterb's rake bootstrapper invoke the gem's own `db:tenant` task when it's defined, so a tenant is set before models are inspected. Apps without the gem are unaffected.

I reproduced the crash in a real Rails 8.1 + activerecord-tenanted app, confirmed the fix there (including `--delete`, the annotate-after-migrate hook, `annotaterb routes` and the `ARTENANT` variable), and ran the full suite.

Known limitation: the gem's task only sets the tenant for its configured `connection_class`, so apps with multiple tenanted database configs would need further support.

## Review

Merged as submitted, with no changes requested. The maintainer approved it and noted he hadn't used activerecord-tenanted himself, but expected no effect on apps that don't use it. That's why the fix only runs the gem's `db:tenant` task when it exists.

## What I learned

- Claim an issue before coding; otherwise someone else may get there first.
- How activerecord-tenanted manages connection pools per tenant, and why code that boots a Rails app needs a tenant selected.
