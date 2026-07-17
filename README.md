<p align="center">
  <img src="https://img.shields.io/badge/Git-Repository%20Migration-F05032?logo=git&logoColor=white" alt="Git repository migration">
  <img src="https://img.shields.io/badge/GitHub-CLI-181717?logo=github&logoColor=white" alt="GitHub CLI">
  <img src="https://img.shields.io/badge/PowerShell-Bulk%20Automation-5391FE?logo=powershell&logoColor=white" alt="PowerShell automation">
</p>

<h1 align="center">GitHub Account-to-Account Repository Migration ETL</h1>

This repository documents a reusable method for migrating GitHub repositories between two accounts controlled by the same person or organisation. The workflow is designed to work with any repository name and can be executed from the Windows command line with **Git** and **GitHub CLI**.

The project treats repository migration as a controlled data-engineering process:

```text
Extract → Validate → Load → Verify → Close
```

A Git mirror is extracted from the source account, validated locally, loaded into a newly created destination repository, and compared against the source before temporary migration data is removed.

The documentation covers:

* migration of one repository through Command Prompt;
* creation of the destination repository without using the GitHub website;
* preservation of branches, tags, commits, files, and Git history;
* destination validation and local clean-up;
* contributor attribution and commit-email behaviour;
* bulk migration of multiple repositories with PowerShell;
* GitHub's native transfer mechanism as an alternative when platform metadata must also be retained.

<h1 align="center"><i>Why Migrate Repositories Between Accounts?</i></h1>

Repository migration may be required when:

* consolidating a personal and professional GitHub presence;
* moving completed portfolio projects to a dedicated business account;
* separating training repositories from production or client-facing work;
* adopting a new professional identity or brand;
* reorganising repositories before a job search or portfolio launch;
* moving projects into an organisation account;
* creating an independently controlled copy before retiring an older account.

A controlled migration prevents accidental loss of history and provides evidence that the destination contains the same branches, tags, and commits as the source.

<h1 align="center"><i>Choose the Correct Migration Strategy</i></h1>

GitHub supports two conceptually different approaches.

| Strategy | Best use | Preserves Git history | Preserves issues, pull requests, stars, settings, and other GitHub metadata |
| --- | --- | ---: | ---: |
| **Native repository transfer** | The repository should change ownership completely. | Yes | Yes, subject to GitHub's transfer rules |
| **Git mirror migration** | A new independent repository should be created, or the process is being used as an ETL exercise or backup. | Yes | No |

## Native transfer

A native transfer is normally preferable when the objective is to change the owner of an existing repository while retaining its GitHub-specific data. GitHub transfers the repository itself rather than creating a separate copy.

For a personal-account-to-personal-account transfer, the target owner must accept the transfer request. The destination account must not already contain a repository with the same name.

## Mirror migration

A mirror migration creates a new repository and copies all Git references held by the source mirror. It preserves:

* files and folders;
* commit history;
* commit authorship metadata;
* branches;
* tags;
* Git notes and other pushable Git references.

It does not automatically reproduce:

* issues and pull requests;
* stars, watchers, and forks;
* releases and release assets;
* repository secrets and variables;
* Actions settings and environments;
* branch-protection rules;
* collaborators and permissions;
* traffic analytics;
* repository discussions, projects, or other platform metadata.

<h1 align="center"><i>ETL Architecture</i></h1>

```text
Source GitHub Account
        │
        │  1. EXTRACT
        ▼
Local Bare Git Mirror
        │
        │  2. VALIDATE
        ▼
Integrity, branches, tags, and commit count
        │
        │  3. LOAD
        ▼
Destination GitHub Account
        │
        │  4. VERIFY
        ▼
Reference comparison and visual inspection
        │
        │  5. CLOSE
        ▼
Normal working clone or secure clean-up
```

<h1 align="center"><i>Prerequisites</i></h1>

The workflow requires:

* Windows 10 or Windows 11;
* access to both GitHub accounts;
* administrator or owner permission for the relevant repositories;
* Git for Windows;
* GitHub CLI;
* enough local disk space to hold the largest repository mirror;
* an email address verified on the destination GitHub account for future commit attribution.

## Install Git

```cmd
winget install --id Git.Git -e --source winget --accept-source-agreements --accept-package-agreements
```

Close and reopen Command Prompt, then verify:

```cmd
git --version
```

## Install GitHub CLI

```cmd
winget install --id GitHub.cli -e --source winget --accept-source-agreements --accept-package-agreements
```

Close and reopen Command Prompt, then verify:

```cmd
gh --version
```

## Authenticate

```cmd
gh auth login --hostname github.com --git-protocol https --web
gh auth status
gh auth setup-git
```

Confirm that the active account is the intended destination account before creating or pushing repositories.

## Configure future commit identity

```cmd
git config --global user.name "YOUR DISPLAY NAME"
git config --global user.email "YOUR_VERIFIED_EMAIL"
```

Confirm the values:

```cmd
git config --global user.name
git config --global user.email
```

The configured email must also be added and verified under the destination account's GitHub email settings.

<h1 align="center"><i>Single-Repository Migration</i></h1>

Replace the placeholders throughout this section:

```text
SOURCE_ACCOUNT
TARGET_ACCOUNT
REPOSITORY_NAME
```

## 1. Create a migration workspace

```cmd
cd /d "%USERPROFILE%\Downloads"
if not exist "GitHub_Repository_Migration" mkdir "GitHub_Repository_Migration"
cd /d "%USERPROFILE%\Downloads\GitHub_Repository_Migration"
```

## 2. Extract the source repository

```cmd
git clone --mirror "https://github.com/SOURCE_ACCOUNT/REPOSITORY_NAME.git"
```

Enter the bare mirror:

```cmd
cd "REPOSITORY_NAME.git"
```

A mirror is not an ordinary editable project directory. It contains Git's object database and references rather than a checked-out working tree.

## 3. Validate the extraction

Check repository integrity:

```cmd
git fsck --full
```

Count commits:

```cmd
git rev-list --all --count
```

List branches and tags:

```cmd
git show-ref
```

Display object statistics:

```cmd
git count-objects -vH
```

Do not continue if Git reports missing or corrupt objects.

## 4. Create the destination repository from Command Prompt

```cmd
gh repo create "TARGET_ACCOUNT/REPOSITORY_NAME" --public
```

For a private repository, use:

```cmd
gh repo create "TARGET_ACCOUNT/REPOSITORY_NAME" --private
```

Do not initialise the destination with a README, licence, or `.gitignore`. The repository should be empty before the mirror push.

## 5. Load the mirror

```cmd
git push --mirror "https://github.com/TARGET_ACCOUNT/REPOSITORY_NAME.git"
```

The command copies the mirror's pushable references, including branches and tags, to the destination.

## 6. Verify the destination

Inspect repository metadata:

```cmd
gh repo view "TARGET_ACCOUNT/REPOSITORY_NAME" --json nameWithOwner,visibility,defaultBranchRef
```

Return to the migration directory:

```cmd
cd ..
```

Create sorted source and destination reference lists:

```cmd
git ls-remote --heads --tags "https://github.com/SOURCE_ACCOUNT/REPOSITORY_NAME.git" | sort > source_refs.txt
git ls-remote --heads --tags "https://github.com/TARGET_ACCOUNT/REPOSITORY_NAME.git" | sort > destination_refs.txt
```

Compare them:

```cmd
fc source_refs.txt destination_refs.txt
```

The preferred result is:

```text
FC: no differences encountered
```

Also inspect the repository in GitHub and confirm that:

* the expected default branch exists;
* the commit count is correct;
* important files open normally;
* the README renders correctly;
* notebooks, images, HTML files, and datasets are readable;
* all expected branches and tags are present.

## 7. Create a normal editable clone

The bare mirror is intended only for migration. Create a normal working copy from the destination:

```cmd
git clone "https://github.com/TARGET_ACCOUNT/REPOSITORY_NAME.git"
cd "REPOSITORY_NAME"
git remote -v
```

Both `origin` entries should point to the destination account.

## 8. Search for obsolete account links

Repository migration does not rewrite URLs embedded in code or documentation.

```cmd
git grep -n "SOURCE_ACCOUNT"
```

Update links where required, then commit and push normally:

```cmd
git add .
git commit -m "Update account and repository links after migration"
git push origin main
```

<h1 align="center"><i>Contributor Attribution</i></h1>

The repository owner and repository contributors are separate concepts.

A mirror migration preserves the existing commit headers. GitHub associates each commit with an account primarily through the email address recorded in that commit. Consequently, commits created under the source account normally remain attributed to the source account after migration.

Changing these local settings:

```cmd
git config --global user.name "NEW NAME"
git config --global user.email "NEW VERIFIED EMAIL"
```

changes **future commits only**. It does not change older commits already contained in the repository history.

## Recommended approach

1. Add and verify the new email address on the destination GitHub account.
2. Keep the historical commits unchanged.
3. Make future commits with the destination account's verified email address.
4. Accept that the source account remains visible as the author of the historical work.

This preserves an accurate and cryptographically stable history. After the destination account makes a new commit, it should also appear in the contributors list, while the original contributor normally remains visible.

## Reassigning historical commits without rewriting history

Inspect the historical author emails:

```cmd
git log --format="%an <%ae>"
```

If the old commits use an ordinary email address that can be moved between the accounts, it may be possible to:

1. remove that email address from the source account;
2. add and verify the exact address on the destination account;
3. wait for GitHub's attribution data to refresh.

An email address can be associated with only one GitHub account at a time. Moving the email also moves the associated historical attribution away from the old account.

This method does not work for an old account-specific GitHub `noreply` address because that address cannot be detached and connected to another account.

## Rewriting historical authorship

Git can rewrite old author and committer metadata, but this is strongly discouraged unless the repository is private, personal, unshared, and fully backed up.

History rewriting:

* creates new commit identifiers for every affected commit;
* requires a force push;
* invalidates existing clones and references to old commit hashes;
* may disrupt forks, pull requests, tags, releases, and signed commits;
* changes the historical record rather than merely changing repository ownership.

For most portfolio migrations, retaining the historical contributor and using the new account for all future commits is the safer and more transparent solution.

<h1 align="center"><i>Bulk Repository Migration</i></h1>

Bulk migration should be tested first with one non-critical repository.

Two bulk approaches are available:

1. **bulk native transfer** through GitHub's repository-transfer API;
2. **bulk mirror duplication** through Git, GitHub CLI, and PowerShell.

## Bulk native transfer

Native transfer is preferable when GitHub metadata must be preserved. A transfer request between two personal accounts must be accepted by the target owner, and the target must not already contain a conflicting repository name.

The transfer endpoint can be called with GitHub CLI:

```powershell
gh api `
    --method POST `
    "repos/SOURCE_ACCOUNT/REPOSITORY_NAME/transfer" `
    -f new_owner="TARGET_ACCOUNT"
```

A controlled PowerShell loop can initiate multiple transfers:

```powershell
$SourceAccount = "SOURCE_ACCOUNT"
$TargetAccount = "TARGET_ACCOUNT"

$repositories = gh repo list $SourceAccount `
    --limit 1000 `
    --source `
    --json name |
    ConvertFrom-Json

foreach ($repository in $repositories) {
    Write-Host "Requesting transfer:" $repository.name

    gh api `
        --method POST `
        "repos/$SourceAccount/$($repository.name)/transfer" `
        -f new_owner="$TargetAccount"

    if ($LASTEXITCODE -ne 0) {
        Write-Warning "Transfer request failed: $($repository.name)"
    }
}
```

Use this only after reviewing GitHub's transfer prerequisites. Personal-account transfers still require target acceptance and are processed asynchronously.

## Bulk mirror-duplication script

The following script:

* lists repositories owned by the source account;
* preserves public or private visibility;
* optionally excludes forks and archived repositories;
* creates one mirror at a time;
* validates the mirror;
* creates the destination repository;
* pushes all Git references;
* compares local and destination branches and tags;
* records the outcome in a CSV manifest;
* skips a repository when the destination name already exists.

Save it as:

```text
migrate_repositories_bulk.ps1
```

```powershell
param(
    [Parameter(Mandatory)]
    [string]$SourceAccount,

    [Parameter(Mandatory)]
    [string]$TargetAccount,

    [string]$Workspace = (
        Join-Path $env:USERPROFILE "Downloads\GitHub-Bulk-Migration"
    ),

    [switch]$IncludeForks,
    [switch]$IncludeArchived
)

$ErrorActionPreference = "Stop"

foreach ($command in @("git", "gh")) {
    if (-not (Get-Command $command -ErrorAction SilentlyContinue)) {
        throw "$command is not installed or is not available through PATH."
    }
}

$runId = Get-Date -Format "yyyyMMdd-HHmmss"
$runDirectory = Join-Path $Workspace $runId
$manifestPath = Join-Path $runDirectory "migration_manifest.csv"

New-Item -ItemType Directory -Path $runDirectory -Force | Out-Null

Write-Host "Switching to source account: $SourceAccount"
gh auth switch --hostname github.com --user $SourceAccount
if ($LASTEXITCODE -ne 0) {
    throw "Unable to activate the source GitHub account."
}

gh auth setup-git
if ($LASTEXITCODE -ne 0) {
    throw "Unable to configure Git authentication."
}

$repositoryJson = gh repo list $SourceAccount `
    --limit 1000 `
    --json name,visibility,isFork,isArchived,description

if ($LASTEXITCODE -ne 0) {
    throw "Unable to list source repositories."
}

$repositories = $repositoryJson | ConvertFrom-Json

if (-not $IncludeForks) {
    $repositories = $repositories | Where-Object { -not $_.isFork }
}

if (-not $IncludeArchived) {
    $repositories = $repositories | Where-Object { -not $_.isArchived }
}

$results = [System.Collections.Generic.List[object]]::new()

foreach ($repository in $repositories) {
    $name = $repository.name
    $mirrorPath = Join-Path $runDirectory "$name.git"
    $sourceUrl = "https://github.com/$SourceAccount/$name.git"
    $destinationUrl = "https://github.com/$TargetAccount/$name.git"

    $status = "failed"
    $message = ""
    $commitCount = 0

    Write-Host ""
    Write-Host "========================================"
    Write-Host "Migrating: $SourceAccount/$name"
    Write-Host "========================================"

    try {
        gh auth switch --hostname github.com --user $SourceAccount
        if ($LASTEXITCODE -ne 0) {
            throw "Could not activate the source account."
        }

        git clone --mirror $sourceUrl $mirrorPath
        if ($LASTEXITCODE -ne 0) {
            throw "Mirror clone failed."
        }

        Push-Location $mirrorPath

        git fsck --full
        if ($LASTEXITCODE -ne 0) {
            throw "Repository integrity check failed."
        }

        $commitCount = [int](git rev-list --all --count)
        if ($LASTEXITCODE -ne 0) {
            throw "Commit count failed."
        }

        gh auth switch --hostname github.com --user $TargetAccount
        if ($LASTEXITCODE -ne 0) {
            throw "Could not activate the target account."
        }

        gh auth setup-git
        if ($LASTEXITCODE -ne 0) {
            throw "Could not configure target Git authentication."
        }

        $null = gh repo view "$TargetAccount/$name" --json name 2>$null
        if ($LASTEXITCODE -eq 0) {
            throw "Destination repository already exists; skipped for safety."
        }

        $visibilityFlag = switch ($repository.visibility) {
            "PRIVATE"  { "--private" }
            "INTERNAL" { "--internal" }
            default    { "--public" }
        }

        $createArguments = @(
            "repo",
            "create",
            "$TargetAccount/$name",
            $visibilityFlag
        )

        if ($repository.description) {
            $createArguments += @(
                "--description",
                [string]$repository.description
            )
        }

        & gh @createArguments
        if ($LASTEXITCODE -ne 0) {
            throw "Destination repository creation failed."
        }

        git push --mirror $destinationUrl
        if ($LASTEXITCODE -ne 0) {
            throw "Mirror push failed."
        }

        $localRefs = git show-ref |
            Where-Object { $_ -match " refs/(heads|tags)/" } |
            Sort-Object

        $remoteRefs = git ls-remote --heads --tags $destinationUrl |
            ForEach-Object { $_ -replace "`t", " " } |
            Sort-Object

        $difference = Compare-Object $localRefs $remoteRefs

        if ($difference) {
            throw "Branch or tag validation failed."
        }

        $status = "success"
        $message = "Migration and reference validation completed."
    }
    catch {
        $message = $_.Exception.Message
        Write-Warning "$name: $message"
    }
    finally {
        if ((Get-Location).Path -eq $mirrorPath) {
            Pop-Location
        }

        $results.Add(
            [pscustomobject]@{
                repository     = $name
                source         = "$SourceAccount/$name"
                destination    = "$TargetAccount/$name"
                visibility     = $repository.visibility
                commit_count   = $commitCount
                status         = $status
                message        = $message
                completed_at   = (Get-Date).ToString("s")
            }
        )

        if (Test-Path $mirrorPath) {
            Remove-Item $mirrorPath -Recurse -Force
        }

        $results | Export-Csv `
            -Path $manifestPath `
            -NoTypeInformation `
            -Encoding UTF8
    }
}

Write-Host ""
Write-Host "Bulk migration completed."
Write-Host "Manifest: $manifestPath"

$results | Format-Table `
    repository,
    visibility,
    commit_count,
    status `
    -AutoSize
```

Before running the script, authenticate both accounts with GitHub CLI so that `gh auth switch` can select either one.

Example execution:

```powershell
Set-ExecutionPolicy -Scope Process Bypass

.\migrate_repositories_bulk.ps1 `
    -SourceAccount "SOURCE_ACCOUNT" `
    -TargetAccount "TARGET_ACCOUNT"
```

To include forks and archived repositories:

```powershell
.\migrate_repositories_bulk.ps1 `
    -SourceAccount "SOURCE_ACCOUNT" `
    -TargetAccount "TARGET_ACCOUNT" `
    -IncludeForks `
    -IncludeArchived
```

The script intentionally skips existing destination repositories rather than overwriting them.

<h1 align="center"><i>Bulk-Migration Limitations</i></h1>

A mirror-based bulk migration copies Git data, not the entire GitHub platform state. After a large migration, separately review:

* repository descriptions, home pages, and topics;
* issues, pull requests, discussions, and projects;
* releases and binary release assets;
* collaborators, teams, and permissions;
* branch protections and rulesets;
* Actions secrets, variables, environments, and runners;
* webhooks, deploy keys, and GitHub Apps;
* Git LFS objects;
* GitHub Pages configuration and custom domains;
* package ownership and container permissions;
* archived status and feature toggles.

Where these items are important, use native repository transfer rather than duplication, or implement a dedicated GitHub API migration for each metadata category.

<h1 align="center"><i>Validation Checklist</i></h1>

A migration should be considered complete only when:

```text
Source branch and tag references
              =
Destination branch and tag references
```

and when all of the following are true:

* the destination repository exists under the correct account;
* the visibility is correct;
* the default branch is correct;
* commit counts match;
* branches and tags match;
* important files render successfully;
* embedded source-account links have been reviewed;
* future Git identity points to a verified destination-account email;
* temporary mirrors have been retained as backups or removed deliberately;
* failed repositories are documented in a manifest and retried individually.

<h1 align="center"><i>Safe Clean-up</i></h1>

After validation, the Command Prompt can be closed immediately. Closing the terminal does not undo or interrupt a completed push.

To retain the local mirror as a temporary backup, simply leave the migration directory in place.

To remove the complete temporary workspace from Command Prompt:

```cmd
cd /d "%USERPROFILE%\Downloads"
rmdir /s /q "GitHub_Repository_Migration"
```

Then close the terminal or run:

```cmd
exit
```

Do not delete a normal editable clone that contains unpushed work.

<h1 align="center"><i>Troubleshooting</i></h1>

## `git` is not recognised

Close and reopen the terminal after installation. If necessary, restart Windows and run:

```cmd
where git
git --version
```

## `gh` is not recognised

Close and reopen Command Prompt, then run:

```cmd
where gh
gh --version
```

The usual installation path is:

```text
C:\Program Files\GitHub CLI\gh.exe
```

## The wrong GitHub account is active

```cmd
gh auth status
gh auth switch --hostname github.com --user TARGET_ACCOUNT
```

## The destination repository already exists

Do not use `git push --mirror` against an unrelated existing repository. Rename the destination, delete the empty mistaken repository, or inspect it carefully before proceeding.

## Authentication fails for a private source repository

Authenticate the source account with GitHub CLI and switch to it before cloning:

```cmd
gh auth switch --hostname github.com --user SOURCE_ACCOUNT
gh auth setup-git
```

Switch to the destination account before creating and pushing the new repository.

## Large files fail to push

Inspect whether the source uses Git LFS and verify that the relevant LFS objects and permissions are available. GitHub also enforces repository and push limits that may require a separate large-file strategy.

## Reference comparison differs

Compare the output of:

```cmd
git show-ref
git ls-remote --heads --tags "DESTINATION_URL"
```

Check for missing branches, tags, protected references, rejected objects, or failed Git LFS transfers.

<h1 align="center"><i>Skills Demonstrated</i></h1>

* Git repository mirroring and duplication.
* GitHub CLI installation and authentication.
* Command-line repository creation.
* ETL-oriented migration design.
* Git object and reference validation.
* Branch and tag comparison.
* Commit-history preservation.
* Contributor-attribution analysis.
* Secure multi-account authentication.
* PowerShell bulk automation.
* CSV migration-manifest generation.
* Retry planning and failure isolation.
* Post-migration link remediation.
* Controlled clean-up and auditability.

<h1 align="center"><i>Technologies Used</i></h1>

<table align="center">
<tr>
<td>
<i>

* Git
* GitHub
* GitHub CLI
* Windows Command Prompt
* PowerShell
* GitHub REST API
* CSV manifests
* ETL methodology

</i>
</td>
</tr>
</table>

<h1 align="center"><i>Overall Summary</i></h1>

This project provides a reusable framework for moving repositories between GitHub accounts through either native ownership transfer or Git mirror duplication.

The mirror workflow is organised as an ETL process: repositories are extracted into local mirrors, validated through Git integrity and reference checks, loaded into newly created destination repositories, and verified through branch, tag, commit, and file inspection. A PowerShell implementation extends the same logic to multiple repositories while preserving visibility and recording a migration manifest.

Contributor attribution is deliberately treated separately from repository ownership. Historical commits remain linked to the account associated with their stored commit email, while future commits can be attributed to the destination account by using an email address verified there. This preserves the integrity of the original history and avoids unnecessary force-pushed rewrites.

<h1 align="center"><i>References</i></h1>

* [GitHub Docs: Duplicating a repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/duplicating-a-repository)
* [GitHub Docs: Importing an external Git repository using the command line](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/importing-an-external-git-repository-using-the-command-line)
* [GitHub Docs: Transferring a repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/transferring-a-repository)
* [GitHub REST API: Transfer a repository](https://docs.github.com/en/rest/repos/repos#transfer-a-repository)
* [GitHub CLI: `gh repo create`](https://cli.github.com/manual/gh_repo_create)
* [GitHub CLI: `gh repo list`](https://cli.github.com/manual/gh_repo_list)
* [GitHub CLI: `gh auth switch`](https://cli.github.com/manual/gh_auth_switch)
* [GitHub Docs: Setting your commit email address](https://docs.github.com/en/account-and-profile/how-tos/email-preferences/setting-your-commit-email-address)
* [GitHub Docs: Why are my commits linked to the wrong user?](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/troubleshooting-commits/why-are-my-commits-linked-to-the-wrong-user)
* [GitHub Docs: Troubleshooting adding an email](https://docs.github.com/en/account-and-profile/how-tos/email-preferences/troubleshooting-adding-an-email)

# Author
# ***[Matteo Meloni](https://www.linkedin.com/in/matteo-meloni-40a357154/)***
