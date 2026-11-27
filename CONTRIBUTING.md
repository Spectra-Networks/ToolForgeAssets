# Contributing to ToolForge Assets

ToolForge Assets is the Spectra Networks technician toolkit. These scripts run on client machines, often unattended and as SYSTEM, in regulated environments such as healthcare, pharmaceutical, biotech, and financial services. Every change should leave a client machine in a known, verifiable state.

## Workflow

- Open an issue, or talk it through with the team first, before starting a new script, a new dependency, an architectural change, or anything that changes how a script treats user data.
- No advance discussion is needed for a focused bug fix that keeps the intended behavior, a regression test, or a documentation correction.
- Address one coherent problem per PR. Keep unrelated formatting, cleanup, file moves, and refactoring in separate PRs.
- Keep behavior outside the scope of the change as it is. Reuse the existing structure and conventions described below.
- Don't change a script's version or `Updated:` date unless the PR is release work.
- Avoid automatic issue-closing keywords (`Fixes #12`) in commits and PR descriptions. Close issues by hand once the fix is confirmed.
- Commit messages are short imperative summaries, usually naming the script: `Fix TICK config exit code`.

## Repository layout

Scripts are grouped by language under `scripts/`. PowerShell scripts live in `scripts/ps/`, with their tests in `scripts/ps/tests/` and their catalog in `scripts/ps/descriptions.json`. When adding a new language or platform, give it its own folder under `scripts/`, keep its tests and metadata inside that folder, and document it in the README.

## Testing and verification

Add a focused regression test for a reproducible bug when there is a useful test boundary. A passing test suite does not prove a script works on a real machine: check the affected workflow in a disposable VM before it reaches a client.

In the PR, state which checks passed and which paths you could not test. Use fabricated or redacted fixtures. Never put client names, hostnames, credentials, recovery keys, or other client data in issues, logs, test files, or commits.

## Dependencies and data safety

Discuss new or replacement dependencies before adding them: why they're needed, their license, maintenance status, size, and security or update impact.

Don't suppress warnings or weaken validation to make a test pass. Keep existing files and user data intact when an operation fails, and call out any data-loss or compatibility risk in the PR.

## PR descriptions

Include:

- The problem and the related issue
- What changed and why
- What machine changes the script makes, and how failures are handled
- How you verified it, and what you couldn't test
- Known limitations

## PowerShell scripts

### Runtime contract

Every script in `scripts/ps/` must keep working on Windows PowerShell 5.1 and in Kaseya LiveConnect and other RMM consoles. That covers `-File`, `-Command`, `-EncodedCommand`, pasted input, running as SYSTEM, and the 32-bit host. Keep each script a single self-contained file with no module dependencies and no syntax newer than 5.1. Prompts must not hang without a console, and an unanswered destructive choice must default to doing nothing.

### Entry point and exit codes

Each script keeps its body in `Invoke-<Name>Main` and ends with:

```powershell
if (-not $global:KillerScriptsTestMode) { Invoke-<Name>Main }
```

Exit through `Exit-KillerScript <code>` instead of calling `exit` directly. In test mode it throws `KILLERSCRIPT_EXIT:<code>` so tests can load a script without running it. (These identifiers come from the upstream project and are kept as-is.)

| Code | Meaning |
| --- | --- |
| 0 | Success |
| 1 | Failure |
| 2 | Not elevated, or a precondition failed |
| 3 | Partial success |
| 3010 | Success, reboot required |

The final status line must agree with the exit code.

### Running the tests

The tests use Pester 5 under both Windows PowerShell 5.1 and PowerShell 7, on Windows. Shipped scripts never load Pester. Run the suite with:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\scripts\ps\tests\Run-Tests.ps1
```

Test files dot-source `scripts\ps\tests\TestHelpers.ps1` and call `Initialize-KillerSafetyNet` in `BeforeAll`. Any destructive cmdlet, native tool, or network call the test hasn't mocked then throws, and file and registry writes are allowed only in `TestDrive` and `TestRegistry`.

### Adding a script

Add the script to `scripts/ps/`, add `scripts/ps/tests/<NAME>.Tests.ps1`, bump the script count in `scripts/ps/tests/Syntax.Tests.ps1`, add an entry to `scripts/ps/descriptions.json`, and add a README section, a Contents row, and update the `Scripts-NN` badge.
