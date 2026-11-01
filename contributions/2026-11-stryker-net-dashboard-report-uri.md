# [fix] Print report URI after uploading the final dashboard report

| | |
|---|---|
| **Project** | [Stryker.NET](https://github.com/stryker-mutator/stryker-net) (mutation testing for .NET) |
| **Pull request** | [stryker-mutator/stryker-net#3939](https://github.com/stryker-mutator/stryker-net/pull/3939) |
| **Type** | `fix` |
| **Stack** | C#, .NET |
| **Status** | ✅ Merged |
| **Date** | 2026-11 |
| **Related issue** | [stryker-mutator/stryker-net#3227](https://github.com/stryker-mutator/stryker-net/issues/3227) |

## Problem

The dashboard reporter has two modes. In real-time mode, the report link is shown as soon as mutants are created. Without real-time reporting, Stryker uploaded the final mutation report to the Stryker Dashboard at the end of the run, but never printed where it went, so users had to find the report themselves.

## How I found it

I was looking for well-scoped issues in C#/.NET projects I'd actually use. Stryker.NET is a mutation testing tool, and #3227 was a small, clearly described bug in the reporting flow: a good size for getting to know an unfamiliar codebase. I reproduced it by running Stryker with the dashboard reporter and real-time reporting off: the report was uploaded, but nothing in the output said where.

## The fix

`DashboardReporter.OnAllMutantsTested` was discarding the result of the upload:

```diff
-        _dashboardClient.PublishReport(mutationReport, _options.ProjectVersion).Wait();
+        var reportUri = _dashboardClient.PublishReport(mutationReport, _options.ProjectVersion).Result;
+
+        // In real-time mode the report link was already shown when the mutants were created
+        if (!publishInRealTime)
+        {
+            OpenDashboardReport(reportUri);
+        }
```

The upload already returned the report URI. The reporter now keeps it and passes it to the existing `OpenDashboardReport` method. That method prints the link, opens it in the browser when `--open-report:dashboard` is set, and logs an error if the upload failed. In real-time mode the link was already shown, so it is skipped there to avoid printing it twice.

Tests added to `DashboardReporterTests`:
- `ShouldPrintReportUriWhenAllMutantsTested`
- `ShouldLogErrorWhenReportUploadFailsAfterAllMutantsTested`
- `ShouldNotPrintReportUriAgainWhenAllMutantsTestedInRealTimeMode`

## Review

Reviewer approved and thanked for it. PR was merged without changes.

## What I learned

- **Read what a method returns before assuming it needs changing.** `PublishReport` already returned the report URI; the caller was just throwing it away with `.Wait()`. The fix was in the caller, not the client.
- **Check every mode a feature runs in.** The link was already printed in real-time mode, so printing it unconditionally would have shown it twice. The third test exists to lock that down.
- **Test the failure path too.** When the upload fails the URI is `null`, and the reporter should log an error instead of printing an empty link. That got its own test.
- **Long reviews mean keeping the branch current.** The PR waited about a week, and `master` was merged into it three times (once by a maintainer) to keep it mergeable.
