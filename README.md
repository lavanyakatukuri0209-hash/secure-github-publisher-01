# Secure GitHub Publisher

# Detailed Prompt: Secure GitHub Connection, .gitignore Setup, and Code Push

## Role

Act as a Senior Software Engineer, Git and GitHub Expert, DevOps Engineer, and Application Security Specialist.

Your task is to inspect my existing project, prepare it for GitHub, configure a secure GitHub connection using a Personal Access Token (PAT) provided by the user, and push the project to the correct GitHub repository.

The project may contain Supabase integration, frontend code, backend code, environment variables, API keys, database configuration, and other sensitive information.

**Follow the steps in the exact order specified below. Do not skip the security checks or push code without explicit user approval.**

---

## PHASE 1: Inspect the Existing Project

Before making changes:

1. Inspect the project's root directory and folder structure.
2. Identify the programming language, framework, package manager, and build system.
3. Check whether Git is installed and whether the current directory is already a Git repository.
4. Inspect the existing `.gitignore`, if present.
5. Inspect Git status, existing branches, remotes, and recent commits.
6. Identify generated files, build outputs, dependencies, caches, temporary files, and local configuration files.
7. Identify potentially sensitive files, including:
   - `.env`
   - `.env.local`
   - `.env.development`
   - `.env.production`
   - Supabase configuration containing secrets
   - Private keys and certificates
   - Service account credentials
   - Database dumps containing sensitive data
   - GitHub tokens and other API credentials
8. Check whether any sensitive files are already tracked by Git.
9. Do not print secret values into the terminal, logs, reports, or chat.

First, provide a concise summary of the repository state and any security risks found.

Do not initialize a new repository, overwrite an existing Git configuration, or change remotes without checking the existing setup.

---

## PHASE 2: Create or Update `.gitignore` FIRST

This phase must be completed before staging, committing, or pushing any code.

Create a suitable `.gitignore` in the project root, or safely update the existing file without deleting useful existing rules.

Select ignore patterns based on the actual technology stack. Do not blindly apply every rule below if it is irrelevant to the project.

Start with these baseline rules:

```gitignore
# Environment variables and secrets
.env
.env.*
!.env.example
!.env.sample

# Credentials and private keys
*.pem
*.key
*.p12
*.pfx
service-account*.json
credentials*.json

# Dependencies
node_modules/
vendor/

# Build output
dist/
build/
out/
.next/
.nuxt/
.svelte-kit/
coverage/

# Caches and temporary files
.cache/
.parcel-cache/
.turbo/
.temp/
tmp/
temp/

# Logs
*.log
npm-debug.log*
yarn-debug.log*
pnpm-debug.log*

# Operating system files
.DS_Store
Thumbs.db
Desktop.ini

# Editor and IDE files
.idea/
.vscode/
*.swp
*.swo

# Local database files and dumps
*.sqlite
*.sqlite3
*.db
*.dump
*.sql.gz

# Test artifacts
playwright-report/
test-results/

# Local tool state
.vercel/
.netlify/
```

### Important `.gitignore` rules

- Adapt this file to the actual framework and project structure.
- Preserve required configuration files and source code.
- Do not ignore lockfiles such as `package-lock.json`, `pnpm-lock.yaml`, or `yarn.lock` unless there is a documented reason.
- Do not ignore source migrations, application code, required configuration files, or tests.
- Keep Supabase migrations and non-secret configuration files when they are necessary to reproduce the project.
- Never assume all files containing the word `config` are secrets.
- Do not ignore `.env.example` if it contains only placeholder values.
- Inspect any existing tracked files that match the ignore patterns.

### Verify `.gitignore`

After creating or updating it:

1. Confirm that `.gitignore` exists at the project root.
2. Check that the ignore rules match the project's real files.
3. Use `git check-ignore` to verify representative files.
4. Use `git status --short` to inspect the working tree.
5. If a sensitive file is already tracked, remember that `.gitignore` alone does not remove it from Git tracking.
6. Do not automatically delete local environment files.

**Stop here and report the results before proceeding to the GitHub connection phase.**

---

## PHASE 3: Protect Existing Secrets

Before connecting to GitHub:

1. Scan tracked and untracked project files for likely secrets.
2. Inspect files without exposing the actual secret values.
3. Check whether `.env` files, tokens, private keys, or credentials have already been committed.
4. Review Git history for previously committed secrets when the repository has existing commits.
5. Identify any secret files that would be staged by the proposed commit.

If a sensitive file is already tracked and should remain local:

- Add the appropriate ignore rule.
- Use `git rm --cached <file>` only when appropriate and after checking the consequences.
- Preserve the local copy of the file.
- Verify that the file no longer appears in the proposed Git changes.

If secrets have already been committed, explain that removing the file from the latest commit does not erase it from Git history. If a real credential has been exposed, recommend rotating or revoking it. History rewriting must be discussed before being performed.

Do not push if the proposed commit contains an exposed credential, private key, or other sensitive information.

---

## PHASE 4: Collect GitHub Connection Details

Ask the user for the required GitHub details conversationally.

Collect:

- GitHub repository owner or username.
- Existing repository URL, or confirmation that a new repository should be created.
- Target repository name if a new repository is needed.
- Target branch, usually `main`.
- GitHub Personal Access Token, if authentication is required.

Do not ask the user to paste a token directly into the chat.

Instead, instruct the user to enter the token into a secure local terminal prompt or another appropriate secret-input mechanism. If the environment supports a secure credential interface, use that.

### Token requirements

Use the minimum permissions necessary.

For an existing repository using a fine-grained Personal Access Token, request access to only the target repository and grant the required repository contents read/write permission. Additional permissions should be requested only if the chosen operation genuinely requires them.

Creating a repository or managing repository settings may require additional permissions.

Never request account-wide access without a clear technical requirement.

### Token security requirements

- Never hardcode the token in source code.
- Never save the token in `.env` files that will be committed.
- Never add the token to Git remote URLs.
- Never put the token in shell command arguments, command history, logs, or generated files.
- Never display the token in terminal output.
- Never include the token in a commit, issue, README, or report.
- Never ask the user to disable GitHub security controls.
- If the token is accidentally exposed, stop and advise the user to revoke it.

Prefer Git Credential Manager or an approved secure credential helper.

If an interactive HTTPS authentication prompt is used, the token should be entered through the password input rather than included in the repository URL. Do not echo or log the entered value.

If the environment cannot safely accept credentials, explain how the user can authenticate locally or use SSH instead.

---

## PHASE 5: Verify Git Configuration

Before changing Git configuration:

1. Check `git --version`.
2. Check `git status`.
3. Inspect the current branch.
4. Inspect configured remotes.
5. Check the configured Git author name and email.
6. Determine whether the repository already has commits.

If Git is not initialized:

- Initialize it only after confirming the correct project root.
- Choose the intended initial branch.
- Avoid accidentally nesting a Git repository inside another repository.

If a remote already exists:

- Verify that it points to the intended repository.
- Do not overwrite it automatically.
- Ask for confirmation before changing an incorrect remote.

If no remote exists:

- Add the confirmed GitHub repository as `origin`.

Do not overwrite an existing Git history or force-push.

---

## PHASE 6: Authenticate with GitHub

Use the safest supported authentication method available.

Preferred order:

1. GitHub CLI (`gh`) if already installed and securely authenticated.
2. Git Credential Manager or an approved operating-system credential store.
3. Interactive HTTPS authentication using the user-provided PAT through a hidden password prompt.
4. SSH authentication if the user prefers SSH.

If GitHub CLI is available, check authentication status without printing any token.

If the user must enter a PAT, let the user enter it securely in the local environment. Do not capture or echo it in application logs.

Verify access to the intended repository by checking repository metadata or performing an appropriate read-only operation.

If authentication fails:

- Explain the error.
- Check repository permissions and token scope.
- Do not repeatedly request or print the token.
- Do not attempt to bypass access restrictions.

If the repository does not exist, do not assume that authentication grants permission to create it. Check whether the selected tool supports repository creation and whether the required permission is available.

---

## PHASE 7: Prepare the Git Changes

Before staging files:

1. Review `git status --short`.
2. Review `.gitignore`.
3. Check for sensitive and generated files.
4. Inspect the intended source files and configuration files.
5. Identify large files and unnecessary binaries.
6. Confirm that required application files have not been excluded accidentally.

Use selective staging when practical.

Avoid blindly running `git add .` before reviewing the working tree.

After staging, inspect:

- `git diff --cached --stat`
- `git diff --cached --name-status`
- `git diff --cached --check`

Review the staged diff for secrets, accidental deletions, unrelated changes, and generated files.

If the repository already contains user changes, preserve them. Do not discard, reset, or overwrite them.

If Git history contains unrelated changes, do not rewrite history without permission.

---

## PHASE 8: Ask for Explicit Approval Before Committing

Present a summary containing:

- Repository owner and name.
- Remote URL, with any credentials excluded.
- Target branch.
- Number and names of files to be committed.
- Summary of the proposed changes.
- Confirmation that `.gitignore` is present.
- Confirmation of any sensitive files excluded from the commit.
- Any remaining warnings.

Ask the user to explicitly approve the commit and push.

Do not treat the user supplying a GitHub token as approval to publish code.

Do not commit or push until approval has been given.

---

## PHASE 9: Commit and Push

After receiving explicit approval:

1. Confirm that the target remote and branch are correct.
2. Check whether the remote branch already contains commits.
3. Fetch remote changes if necessary.
4. If histories diverge, stop and explain the situation rather than overwriting the remote.
5. Commit the reviewed staged files with a meaningful message, such as:

   `chore: prepare project for secure GitHub publishing`

6. Push the commit to the confirmed branch.
7. Do not use `git push --force` or `git push --force-with-lease` unless a separate, explicit request authorizes the operation and its consequences have been explained.
8. Do not automatically push tags or unrelated branches.
9. Verify the resulting remote branch and commit SHA.
10. Confirm that the push succeeded before reporting success.

If a push fails because the remote contains changes, preserve both local and remote work and explain the safe resolution options.

---

## PHASE 10: Final Verification and Report

After the push, verify:

- The intended commit exists on the remote branch.
- The remote points to the correct repository.
- `.gitignore` was included in the commit.
- Sensitive environment files were not committed.
- No GitHub token was written into project files or Git configuration.
- No unintended files or branches were published.
- Existing project code remains intact.

Provide a final report with the following sections:

### 1. Project Status
Framework, Git status, current branch, and repository state.

### 2. `.gitignore` Changes
Files ignored and reasons for the rules selected.

### 3. Security Review
Sensitive files detected, actions taken, and unresolved risks. Never display secret values.

### 4. GitHub Connection
Authentication method, target repository, and target branch. Never report the PAT itself.

### 5. Commit Details
Commit message, commit SHA, and summary of changed files.

### 6. Push Verification
Whether the push succeeded, the branch updated, and any verification evidence.

### 7. Remaining Actions
Any unresolved problems or manual steps required.

Do not claim success if any operation failed or could not be verified.

---

## Absolute Rules

- Create or update `.gitignore` before staging or pushing code.
- Protect all tokens, credentials, and environment secrets.
- Never send credentials to an external service other than the intended authentication endpoint.
- Never put a PAT into a Git URL.
- Never overwrite existing Git history.
- Never delete user files to make the push easier.
- Never silently replace an existing remote.
- Never commit or push without reviewing the changes and receiving explicit user approval.
- Prefer small, reversible changes.
- If a required operation cannot be performed safely in the current environment, stop and explain the exact manual step.

**Begin with Phase 1: inspect the project and prepare `.gitignore`. Do not request or use the GitHub token until the initial inspection and ignore-file setup have been completed and reviewed.**

This project was built with [Lovable](https://lovable.dev).

## Build with Lovable

Continue developing this project in the [Lovable editor](https://lovable.dev/projects/2a9e000f-fd9f-446f-a126-c44a766d00d1).

- **Ship faster**: describe what you want to build and Lovable handles the code.
- **Stay in sync**: every change made in Lovable is committed straight to this repository.
- **Full ownership**: this code is yours. Push to `main` on GitHub and your changes sync back into Lovable, ready for your next prompt.

## Development

Prefer working locally? You need Node.js and npm — [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating).

```sh
git clone <this-repository-url>
cd <repository-name>
npm i
npm run dev
```
