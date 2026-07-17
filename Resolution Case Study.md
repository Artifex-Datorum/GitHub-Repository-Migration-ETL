<h1 align="center">GitHub Account-to-Account Repository Migration Resolution Case Study</h1>

<h1 align="center">Exercise 1: Understand the migration objective</h1>

The objective is to migrate one or more repositories from a source GitHub account to a destination GitHub account controlled by the same person or organisation while preserving the repository's Git history and maintaining a verifiable audit trail.

The technical workflow is:

```text
Extract → Validate → Load → Verify → Close
```

| Phase | Purpose |
| --- | --- |
| `Extract` | Create a complete local Git mirror of the source repository. |
| `Validate` | Check object integrity, branches, tags, and commit count. |
| `Load` | Create an empty destination repository and mirror-push the Git references. |
| `Verify` | Compare source and destination references and inspect important files. |
| `Close` | Create a normal working clone or remove the temporary mirror. |

This procedure creates an independent copy. It is different from GitHub's native repository-transfer feature, which changes ownership and preserves GitHub-specific metadata such as issues, pull requests, stars, watchers, settings, and redirects.

<h1 align="center">Exercise 2: Select the correct migration method</h1>

Before running commands, determine whether the intended result is a **transfer** or a **duplicate**.

## Native transfer

Use native transfer when:

* the source repository should cease to be owned by the old account;
* issues and pull requests must remain attached;
* stars, watchers, releases, projects, and settings must be retained;
* redirects from the old repository URL are desirable;
* the complete GitHub repository identity should move.

## Mirror migration

Use mirror migration when:

* an independent copy is required;
* the source repository should remain available;
* the operation is being documented as a data-migration exercise;
* a local backup is desirable;
* only Git-controlled content needs to be migrated;
* the repository has little or no important GitHub-platform metadata.

For a portfolio repository with files, notebooks, HTML reports, branches, tags, and commit history—but no important issues or pull requests—the mirror approach is often sufficient.

<h1 align="center">Exercise 3: Install Git</h1>

Open Command Prompt and check whether Git is installed:

```cmd
git --version
```

If Git is unavailable, install it with WinGet:

```cmd
winget install --id Git.Git -e --source winget --accept-source-agreements --accept-package-agreements
```

Close and reopen Command Prompt, then verify:

```cmd
git --version
```

A successful output resembles:

```text
git version 2.x.x.windows.x
```

<h1 align="center">Exercise 4: Install GitHub CLI</h1>

Git operates on repository data. GitHub CLI creates and manages repositories on GitHub from the terminal.

Install it:

```cmd
winget install --id GitHub.cli -e --source winget --accept-source-agreements --accept-package-agreements
```

Close and reopen Command Prompt, then run:

```cmd
gh --version
```

If `gh` remains unavailable, locate it:

```cmd
where /r "C:\Program Files" gh.exe
```

The common installation path is:

```text
C:\Program Files\GitHub CLI\gh.exe
```

<h1 align="center">Exercise 5: Authenticate both GitHub accounts</h1>

Authenticate the destination account:

```cmd
gh auth login --hostname github.com --git-protocol https --web
```

Confirm the active account:

```cmd
gh auth status
```

Configure Git to use GitHub CLI credentials:

```cmd
gh auth setup-git
```

For bulk migration of private repositories, authenticate both accounts. GitHub CLI can store more than one account and switch between them:

```cmd
gh auth login --hostname github.com --git-protocol https --web
gh auth switch --hostname github.com --user SOURCE_ACCOUNT
gh auth switch --hostname github.com --user TARGET_ACCOUNT
```

Always run `gh auth status` before a destructive or ownership-sensitive operation.

<h1 align="center">Exercise 6: Configure future commit identity</h1>

Set the identity for future commits created on the computer:

```cmd
git config --global user.name "YOUR DISPLAY NAME"
git config --global user.email "YOUR_VERIFIED_EMAIL"
```

Verify:

```cmd
git config --global user.name
git config --global user.email
```

The email must be added and verified in the destination GitHub account's email settings. The configured name is descriptive, but GitHub primarily uses the commit email to connect a commit to an account.

This configuration does not change historical commits.

<h1 align="center">Exercise 7: Define the generic migration variables</h1>

The reusable placeholders are:

```text
SOURCE_ACCOUNT
TARGET_ACCOUNT
REPOSITORY_NAME
```

Example conceptual mapping:

```text
Source URL:
https://github.com/SOURCE_ACCOUNT/REPOSITORY_NAME

Destination URL:
https://github.com/TARGET_ACCOUNT/REPOSITORY_NAME
```

The same procedure works when the destination repository uses a different name; replace only the destination name where appropriate.

<h1 align="center">Exercise 8: Create the migration workspace</h1>

Move to Downloads:

```cmd
cd /d "%USERPROFILE%\Downloads"
```

Create a dedicated directory:

```cmd
if not exist "GitHub_Repository_Migration" mkdir "GitHub_Repository_Migration"
```

Enter it:

```cmd
cd /d "%USERPROFILE%\Downloads\GitHub_Repository_Migration"
```

Confirm the current location:

```cmd
cd
```

<h1 align="center">Exercise 9: Extract the source repository</h1>

Run:

```cmd
git clone --mirror "https://github.com/SOURCE_ACCOUNT/REPOSITORY_NAME.git"
```

Expected output resembles:

```text
Cloning into bare repository 'REPOSITORY_NAME.git'...
remote: Enumerating objects...
Receiving objects: 100% ...
Resolving deltas: 100% ...
```

Enter the mirror:

```cmd
cd "REPOSITORY_NAME.git"
```

The `.git` directory is a bare repository. It is not intended for editing files directly.

<h1 align="center">Exercise 10: Validate repository integrity</h1>

Run the complete integrity check:

```cmd
git fsck --full
```

Serious errors such as missing or corrupt objects must be resolved before the migration continues.

Count all reachable commits:

```cmd
git rev-list --all --count
```

List references:

```cmd
git show-ref
```

Display object statistics:

```cmd
git count-objects -vH
```

Record the commit count and any branches or tags that must appear in the destination.

<h1 align="center">Exercise 11: Create the destination repository from Command Prompt</h1>

Make sure GitHub CLI is using the destination account:

```cmd
gh auth switch --hostname github.com --user TARGET_ACCOUNT
gh auth status
```

Create a public repository:

```cmd
gh repo create "TARGET_ACCOUNT/REPOSITORY_NAME" --public
```

Create a private repository instead:

```cmd
gh repo create "TARGET_ACCOUNT/REPOSITORY_NAME" --private
```

Do not add a README, licence, or `.gitignore`. The destination must be empty before the mirror is loaded.

<h1 align="center">Exercise 12: Load the repository</h1>

Run:

```cmd
git push --mirror "https://github.com/TARGET_ACCOUNT/REPOSITORY_NAME.git"
```

A successful output resembles:

```text
To https://github.com/TARGET_ACCOUNT/REPOSITORY_NAME.git
 * [new branch]      main -> main
```

Additional lines will appear for other branches and tags.

A mirror push should be used only with the intended empty destination repository. It can make destination references match the local mirror and is therefore inappropriate for an unrelated established repository.

<h1 align="center">Exercise 13: Verify repository metadata</h1>

Query the destination:

```cmd
gh repo view "TARGET_ACCOUNT/REPOSITORY_NAME" --json nameWithOwner,visibility,defaultBranchRef
```

Confirm:

* correct owner;
* correct repository name;
* correct visibility;
* correct default branch.

Open a terminal view of the repository:

```cmd
gh repo view "TARGET_ACCOUNT/REPOSITORY_NAME"
```

<h1 align="center">Exercise 14: Compare source and destination references</h1>

Return to the parent directory:

```cmd
cd ..
```

Create a source-reference file:

```cmd
git ls-remote --heads --tags "https://github.com/SOURCE_ACCOUNT/REPOSITORY_NAME.git" | sort > source_refs.txt
```

Create a destination-reference file:

```cmd
git ls-remote --heads --tags "https://github.com/TARGET_ACCOUNT/REPOSITORY_NAME.git" | sort > destination_refs.txt
```

Compare them:

```cmd
fc source_refs.txt destination_refs.txt
```

The ideal result is:

```text
FC: no differences encountered
```

This confirms that the visible branches and tags point to the same commits.

<h1 align="center">Exercise 15: Conduct visual validation</h1>

Open the destination repository and inspect:

* the file and folder structure;
* README rendering;
* notebooks and data files;
* HTML reports;
* images and linked assets;
* commit count;
* branch list;
* tag list;
* language analysis;
* release and package sections where relevant.

A completed command does not replace destination validation.

<h1 align="center">Exercise 16: Create a normal working clone</h1>

The migration mirror should not be used as an ordinary development directory.

Create a normal clone:

```cmd
git clone "https://github.com/TARGET_ACCOUNT/REPOSITORY_NAME.git"
```

Enter it:

```cmd
cd "REPOSITORY_NAME"
```

Confirm the remote:

```cmd
git remote -v
```

Both fetch and push URLs should point to the destination account.

<h1 align="center">Exercise 17: Update embedded links</h1>

Search for old account names:

```cmd
git grep -n "SOURCE_ACCOUNT"
```

Review:

* README links;
* raw-file URLs;
* jsDelivr URLs;
* notebook download links;
* image-hosting links;
* badges;
* documentation references;
* CI/CD configuration;
* package references.

Commit corrections:

```cmd
git add .
git commit -m "Update repository links after account migration"
git push origin main
```

<h1 align="center">Exercise 18: Understand why the old account appears under Contributors</h1>

The contributor list represents historical commit authorship, not repository ownership.

A mirror migration preserves the author name and email recorded in every commit. GitHub links those commits to whichever account owns the corresponding commit email address. Therefore, the old account may remain visible under **Contributors** even though the new account owns the repository.

This is correct behaviour and does not give the old account administrative access to the destination repository.

<h1 align="center">Exercise 19: Attribute future commits to the destination account</h1>

Add and verify the configured email under the destination account's GitHub settings.

Confirm the local value:

```cmd
git config --global user.email
```

Create future commits normally:

```cmd
git add .
git commit -m "Document repository migration"
git push origin main
```

When the commit email is verified on the destination account, the new commit should be linked to that account. The destination account should then also appear as a contributor, while the source account normally remains visible for the older commits.

<h1 align="center">Exercise 20: Investigate historical commit emails</h1>

From a normal working clone, run:

```cmd
git log --format="%an <%ae>"
```

For a unique list in PowerShell:

```powershell
git log --format="%an <%ae>" |
    Sort-Object -Unique
```

This reveals the exact names and emails stored in the repository history.

<h1 align="center">Exercise 21: Reassign historical attribution without rewriting Git history</h1>

When the historical commits use a normal personal or business email address, the address may be moved between GitHub accounts:

1. remove the exact email from the source account;
2. add and verify it on the destination account;
3. allow time for GitHub's attribution data to refresh.

An email address can be associated with only one GitHub account at a time.

This approach moves the historical attribution associated with that email. It should be used only when that outcome is intentional.

It cannot be used when the old commits contain the source account's GitHub-provided `noreply` email because an account-specific `noreply` address cannot be detached and assigned to another account.

<h1 align="center">Exercise 22: Avoid unnecessary history rewriting</h1>

Git can rewrite old author and committer information, but doing so replaces affected commits with new commits.

Consequences include:

* changed commit hashes;
* required force pushes;
* broken links to old commits;
* incompatible existing clones;
* possible disruption to forks and pull requests;
* altered tags and releases;
* invalidated commit signatures;
* loss of the original historical record.

GitHub strongly discourages changing old authorship merely to alter account attribution, particularly in shared repositories.

For a professional portfolio, the recommended result is:

```text
Historical commits → correctly attributed to the original account
Future commits     → attributed to the new professional account
Repository owner   → the new professional account
```

<h1 align="center">Exercise 23: Close the completed terminal session</h1>

When the push and validation have finished, Command Prompt can be closed safely. No background Git process is required to keep the repository online.

Run:

```cmd
exit
```

or close the window normally.

The local mirror remains on disk until deleted.

<h1 align="center">Exercise 24: Remove temporary migration data</h1>

Retain the mirror temporarily when an additional backup is useful.

To remove the complete migration workspace after sign-off:

```cmd
cd /d "%USERPROFILE%\Downloads"
rmdir /s /q "GitHub_Repository_Migration"
```

Do not delete a normal editable clone containing unpushed changes.

<h1 align="center">Exercise 25: Plan a bulk migration</h1>

A bulk process should:

1. list repositories owned by the source account;
2. filter forks, archived repositories, or visibility classes as required;
3. migrate one repository at a time;
4. stop or skip safely when a destination name exists;
5. validate each repository independently;
6. record status, commit count, visibility, and errors in a manifest;
7. remove only temporary mirrors;
8. retry failed repositories individually.

GitHub CLI lists owned repositories with:

```powershell
gh repo list SOURCE_ACCOUNT `
    --limit 1000 `
    --json name,visibility,isFork,isArchived,description
```

<h1 align="center">Exercise 26: Run the reusable bulk mirror script</h1>

Use the PowerShell script documented in `README.md`:

```text
migrate_repositories_bulk.ps1
```

Authenticate both accounts first:

```cmd
gh auth login --hostname github.com --git-protocol https --web
gh auth status
```

Run the script:

```powershell
Set-ExecutionPolicy -Scope Process Bypass

.\migrate_repositories_bulk.ps1 `
    -SourceAccount "SOURCE_ACCOUNT" `
    -TargetAccount "TARGET_ACCOUNT"
```

The script creates a timestamped run directory and a CSV migration manifest.

Expected manifest fields include:

| Field | Meaning |
| --- | --- |
| `repository` | Repository name. |
| `source` | Original account and repository. |
| `destination` | Destination account and repository. |
| `visibility` | Public, private, or internal visibility. |
| `commit_count` | Reachable commit count in the mirror. |
| `status` | Success or failure. |
| `message` | Validation result or error explanation. |
| `completed_at` | Completion timestamp. |

<h1 align="center">Exercise 27: Handle bulk native transfers</h1>

When issues, pull requests, releases, stars, settings, redirects, and other platform metadata matter, use GitHub's native transfer API rather than mirror duplication.

Single transfer request:

```powershell
gh api `
    --method POST `
    "repos/SOURCE_ACCOUNT/REPOSITORY_NAME/transfer" `
    -f new_owner="TARGET_ACCOUNT"
```

For transfers between personal accounts:

* the target receives a confirmation request;
* acceptance is required;
* the invitation expires if not accepted within GitHub's permitted period;
* the destination must not already contain a conflicting repository name;
* the original owner normally remains a collaborator after transfer.

Bulk API initiation is possible, but each transfer must still satisfy GitHub's transfer rules and may require target acceptance.

<h1 align="center">Exercise 28: Review metadata not copied by mirror migration</h1>

After a mirror-based bulk migration, inspect separately:

* issues and pull requests;
* releases and release assets;
* repository topics and descriptions;
* branch protections and rulesets;
* Actions workflows, secrets, environments, and runners;
* collaborators and teams;
* deploy keys and webhooks;
* discussions and projects;
* stars, watchers, and forks;
* GitHub Pages and custom domains;
* packages and container images;
* Git LFS objects;
* archive status and feature settings.

A mirror copy should not be described as a complete GitHub-platform migration unless these categories have also been addressed.

<h1 align="center">Exercise 29: Troubleshooting</h1>

## `gh` is not recognised after installation

Close and reopen Command Prompt:

```cmd
where gh
gh --version
```

## Wrong active account

```cmd
gh auth status
gh auth switch --hostname github.com --user TARGET_ACCOUNT
```

## Destination already exists

Do not overwrite it blindly. Inspect the repository, rename the destination, or delete only an empty mistaken repository.

## Private source cannot be cloned

Activate the source account:

```cmd
gh auth switch --hostname github.com --user SOURCE_ACCOUNT
gh auth setup-git
```

## Push rejected

Check:

* destination permissions;
* repository visibility and account plan;
* large-file limits;
* Git LFS configuration;
* protected or hidden references;
* network stability;
* whether the destination was really empty.

## Contributor remains the old account

This is expected when historical commits retain an email associated with the old account. Configure and verify the destination account's email for future commits. Do not rewrite history solely for cosmetic reasons.

<h1 align="center">Exercise 30: Overall resolution</h1>

The migration is complete when:

```text
Source Git references
        =
Destination Git references
```

and all of the following conditions are true:

* the destination repository has the correct owner and visibility;
* the default branch is correct;
* commit counts, branches, and tags match;
* important files open successfully;
* old account links have been reviewed;
* future commit identity uses an email verified on the destination account;
* contributor attribution has been understood and accepted or deliberately reassigned through email ownership;
* temporary mirrors have been retained or deleted intentionally;
* any missing GitHub-platform metadata has been migrated separately or documented as out of scope;
* bulk failures have been captured in a manifest and retried independently.

The resulting workflow is reusable for individual repositories, portfolio consolidation, professional-account migration, organisation onboarding, repository backup, and controlled bulk duplication.

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
