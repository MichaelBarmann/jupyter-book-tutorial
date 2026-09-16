# From Markdown to a Published Jupyter Book

A practical Windows tutorial using Typora, Anaconda Prompt, Git, GitHub Actions, and GitHub Pages.

Based on the setup completed on September 16, 2026. The working local versions were **Jupyter Book 2.1.5** and **Node.js v24.17.0**. This guide records that successful setup; it is not a recommendation to reinstall or upgrade a working environment.

Each numbered section groups related steps and can become a separate Jupyter Book page. Section 12 explains how to split this guide and add it to the existing book.

## Contents

1. [Understand the workflow](#1-understand-the-workflow)
2. [Find or clone the repository](#2-find-or-clone-the-repository)
3. [Find the working Jupyter Book environment](#3-find-the-working-jupyter-book-environment)
4. [Write the pages in Typora](#4-write-the-pages-in-typora)
5. [Configure the book](#5-configure-the-book)
6. [Preview the book locally](#6-preview-the-book-locally)
7. [Configure automatic publishing](#7-configure-automatic-publishing)
8. [Commit, push, and check deployment](#8-commit-push-and-check-deployment)
9. [Add one-click access from GitHub](#9-add-one-click-access-from-github)
10. [Return to the project and make updates](#10-return-to-the-project-and-make-updates)
11. [Troubleshooting](#11-troubleshooting)
12. [Use this guide as book pages](#12-use-this-guide-as-book-pages)
13. [Command reference and links](#13-command-reference-and-links)

## 1. Understand the workflow

### What we built

We created a two-page Jupyter Book inside an existing GitHub repository. We edited ordinary Markdown files with Typora, previewed the complete book in a browser, and configured GitHub to publish changes automatically after a push.

The final locations were:

| Item | Location |
| --- | --- |
| GitHub repository | [MichaelBarmann/jupyter-book-tutorial](https://github.com/MichaelBarmann/jupyter-book-tutorial) |
| Published book | [Jupyter Book Tutorial](https://michaelbarmann.github.io/jupyter-book-tutorial/) |
| Local repository | `C:\Users\mbarm\Documents\Documents\01_GradSchool\03_code\github_repos\jupyter-book-tutorial` |
| Publishing runs | [Actions](https://github.com/MichaelBarmann/jupyter-book-tutorial/actions) |
| Hosting configuration | [Settings → Pages](https://github.com/MichaelBarmann/jupyter-book-tutorial/settings/pages) |

Although we initially discussed putting the book in a `book/` subfolder, the repository already contained a book configuration at its root. **Our actual setup keeps `myst.yml`, `index.md`, and `chapter1.md` at the repository root.** Use this layout when following the commands below.

### Which tool does what?

| Tool or file | Role |
| --- | --- |
| Typora | Edits `.md` source files and previews ordinary Markdown and math. |
| Jupyter Book | Builds those source files into a navigable website. |
| MyST | The Markdown document system used by Jupyter Book 2. |
| `myst.yml` | Configures the book title, table of contents, theme, and footer. |
| Anaconda Prompt | Supplies the environment in which our Jupyter Book installation works. |
| Node.js | Supports the website tooling used by the local preview. |
| Git | Records local changes and transfers commits between the computer and GitHub. |
| GitHub repository | Stores the source files and their version history online. |
| GitHub Actions | Runs the automated build and deployment workflow. |
| GitHub Pages | Hosts the published website. |

A **Jupyter Book** is not the same as a **Jupyter notebook**. A notebook is usually an `.ipynb` file with code cells and outputs. Our book uses `.md` pages; it does not require opening JupyterLab or creating a notebook.

### The four separate actions to remember

1. **Save** in Typora: update a file on your computer.
2. **Commit** in Git: record a version of selected changes locally.
3. **Push**: upload those commits to GitHub.
4. **Deploy**: GitHub Actions builds the website and publishes it to GitHub Pages.

Saving alone does not update GitHub. Committing alone does not update GitHub. Pushing starts the configured publishing workflow; the website changes after that workflow succeeds.

### Local preview versus the public website

- `http://localhost:3000` is a preview served by your own computer. It works while the local preview process is running.
- `https://michaelbarmann.github.io/jupyter-book-tutorial/` is the published website. It stays online when you close the terminal or turn off your computer.

Typora shows the page content, but it does not reproduce the full book theme, navigation, or every MyST feature. Use the browser preview to check the actual book.

### How to read the command blocks

Copy only the command, not a prompt such as `PS ...>` or `(base) C:\Users\mbarm>`.

- Blocks labeled `powershell` belong in **PowerShell**.
- Blocks labeled `bat` belong in **Anaconda Prompt**, which used Windows command-prompt syntax in our session.
- Blocks labeled `markdown` are file contents, not terminal commands.
- Blocks labeled `yaml` belong in `myst.yml`, not in the terminal.

Paths below reproduce this project. Substitute your own path for another project. Keep quotation marks around paths, especially if they contain spaces.

## 2. Find or clone the repository

### Step 1: Check whether you already have a local copy

The repository existed on GitHub, but we were not sure whether it had been cloned onto the computer. Cloning creates a local Git repository, including its tracked files and history, and records the GitHub address as a remote.

In PowerShell, move to the folder where you keep local GitHub repositories:

```powershell
cd "C:\Users\mbarm\Documents\Documents\01_GradSchool\03_code\github_repos"
```

`cd` changes the current directory. It does not create a folder or download anything.

List the immediate subfolders:

```powershell
Get-ChildItem -Directory | Select-Object -ExpandProperty Name
```

- `Get-ChildItem` lists items in the current directory.
- `-Directory` includes only folders.
- `|` sends the result to the next command.
- `Select-Object -ExpandProperty Name` prints only each folder's name.

Our list contained `diff-eq-models`, `diff-eq-tools`, `git-notes`, `git-tutorial`, `particle-sim`, and `projects`. There was no folder named `jupyter-book-tutorial`.

A local repository can have a different name from its GitHub repository. To check beneath the current folder, we ran this read-only search:

```powershell
Get-ChildItem -Force -Recurse -Filter .git -ErrorAction SilentlyContinue |
    ForEach-Object {
        $folder = $_.Parent.FullName
        $remote = git -C "$folder" remote get-url origin 2>$null
        if ($remote -like "*jupyter-book-tutorial*") {
            "$folder → $remote"
        }
    }
```

This was the exact search used during our session. It looks for `.git` entries recursively, obtains the containing folder, asks Git for its `origin` address, and prints matches.

- `-Force` includes hidden entries such as `.git`.
- `-Recurse` searches subfolders.
- `-Filter .git` looks for Git metadata entries.
- `ForEach-Object` processes each result.
- `git -C` runs Git as though the specified folder were the current directory.
- `remote get-url origin` reads the address of the remote named `origin`.
- `2>$null` suppresses error output for that Git call.
- `-like` performs a wildcard match.

**Scope:** no output means this search found no matching `origin` beneath the current folder. It does not prove that no copy exists elsewhere on the computer. This exact script assumes the ordinary `.git` directory layout; linked worktrees and submodules can use a `.git` file and need different path handling.

If you already know a candidate folder, a simpler check is:

```powershell
cd "C:\path\to\candidate-repository"
git remote -v
```

`git remote -v` lists remote names and their fetch/push URLs. It verifies which online repository the local copy uses. GitHub's website does not maintain a list of your local clone paths.

### Step 2: Clone the repository

Our search returned no match. From the parent `github_repos` folder, we ran:

```powershell
git clone https://github.com/MichaelBarmann/jupyter-book-tutorial.git
```

Git automatically created a folder called `jupyter-book-tutorial`. We did not create that folder first, run `git init`, or download a ZIP.

A successful clone printed messages about receiving objects and resolving deltas, then returned to the prompt.

Earlier in our Git discussion, we also used this example:

```powershell
git clone https://github.com/MichaelBarmann/tutorials.git git-tutorial
```

The optional final argument selects the local folder name. That example clones a **different repository** and is not needed for the Jupyter Book setup. It illustrates that local names and GitHub names need not match.

### Step 3: Inspect the cloned repository

```powershell
cd jupyter-book-tutorial
Get-ChildItem -Force
```

`-Force` makes hidden entries visible. Our starting contents were:

| Entry | Meaning |
| --- | --- |
| `.git/` | Git metadata; do not edit it manually. |
| `_site/` | In this project, a source folder containing a custom sidebar-footer file. |
| `.gitignore` | Patterns telling Git which untracked files to ignore. |
| `create_jupyter_book_tutorial.md` | An existing tutorial document. |
| `index.md` | An existing introductory page. |
| `myst.yml` | An existing MyST configuration. |

We inspected the text files with:

```powershell
Get-Content myst.yml
Get-Content index.md
Get-Content .gitignore
```

`Get-Content` displays a text file without modifying it.

The original table of contents listed only `create_jupyter_book_tutorial.md`, and the title was `Git Tutorial`. The original `index.md` also contained escaped Markdown, such as `\# Git Tutorial` and `\- Git basics`. The backslashes made these symbols literal rather than formatting markers.

The existing `.gitignore` contained:

```text
_build/
.ipynb_checkpoints/
.DS_Store
```

We retained it. `_build/` is generated output and caches, `.ipynb_checkpoints/` holds notebook checkpoints, and `.DS_Store` is macOS metadata. Ignore rules do not automatically untrack files that were already committed.

**Do not confuse `_site/` with `_build/` in this project.** Our `_site/primary_sidebar_footer.md` is an intentional source file referenced by the configuration.

## 3. Find the working Jupyter Book environment

### Step 1: Check available commands

Initially, in PowerShell, we ran:

```powershell
jupyter book --version
node --version
myst --version
```

Each `--version` option asks a program to report its installed version. These commands do not install or upgrade anything.

Our results were:

| Command | Result |
| --- | --- |
| `jupyter book --version` | `jupyter` was not recognized. |
| `node --version` | `v24.17.0` |
| `myst --version` | `myst` was not recognized. |

A command not being recognized means that shell cannot find it. It does **not** necessarily mean the software is absent from the computer. Anaconda environments can make commands available in one shell but not another.

### Step 2: Open Anaconda Prompt

Because Jupyter Book had apparently been installed through Anaconda before, we opened **Anaconda Prompt** from the Windows Start menu and ran:

```bat
jupyter book --version
```

It returned:

```text
v2.1.5
```

The prompt began with `(base)`, indicating that the base Conda environment was active. **We did not install Jupyter Book or Node.js during this setup.** We found an existing working installation and used it.

For this computer, start with Anaconda Prompt when returning to the project. A separate working `myst` command is not required for the `jupyter book` commands used here.

If setting up a different computer, use the current [Jupyter Book installation instructions](https://jupyterbook.org/stable/get-started/install/) rather than assuming these packages are already installed. This tutorial documents a Jupyter Book 2 setup; older Jupyter Book 1 tutorials use different configuration files and commands.

### Step 3: Navigate to the repository in Anaconda Prompt

```bat
cd /d "C:\Users\mbarm\Documents\Documents\01_GradSchool\03_code\github_repos\jupyter-book-tutorial"
```

In Windows command-prompt syntax, `/d` lets `cd` change both the drive and the directory. Use it in Anaconda Prompt, not in PowerShell; PowerShell uses `cd "path"` without `/d`.

The prompt should now end with `jupyter-book-tutorial>`. All subsequent Jupyter Book and Git publishing commands are run from this folder, which contains `myst.yml`.

## 4. Write the pages in Typora

### Step 1: Replace the introductory page

Open `index.md` in Typora through **File → Open**. Press **Ctrl+/** to switch to Source Code Mode. Replace its contents with:

```markdown
# Jupyter Book Tutorial

Welcome to my first Jupyter Book.

This book has two short pages.
```

Press **Ctrl+/** again to return to the formatted view, then **Ctrl+S** to save.

The `#` creates a top-level heading. Blank lines separate paragraphs. Source Code Mode is useful when pasting an entire raw Markdown snippet: it avoids having the paste interpreted as literal text or rich formatting. For ordinary writing afterward, use the formatted editor.

Do not copy the surrounding triple-backtick fence into the page. The fence marks the example in this guide; it is not part of the desired page.

We initially suggested this command as an alternative:

```bat
notepad index.md
```

It opens the file in Notepad. We switched to Typora because learning that editor was part of the goal. Notepad is not required for Markdown pages.

### Step 2: Add display mathematics

Typing `$$x^2$$` on one line in the formatted editor did not produce the desired math block. The successful approach was:

1. On a blank line, type `$$` and press **Enter**.
2. In the resulting math block, type `x^2`.
3. Press **Ctrl+Enter** to finish editing the block.
4. Save the file.

The underlying Markdown is:

```markdown
$$
x^2
$$
```

Thus our introductory page ultimately also included this equation below its paragraphs.

For math inside a sentence, use `$x^2$`. First enable **File → Preferences → Markdown → Inline Math**, then restart Typora. These are Typora settings, not edits to `myst.yml`. [Typora math documentation](https://support.typora.io/Math/)

### Step 3: Create the second page

Press **Ctrl+N** in Typora, switch to Source Code Mode, and enter:

```markdown
# My First Chapter

This is the second page of my Jupyter Book.

Here is a simple equation:

$$
f(x) = x^2
$$
```

Return to the formatted view and save as **`chapter1.md`** in the repository root, alongside `index.md` and `myst.yml`.

Check that the filename is exactly `chapter1.md`, not `chapter1.md.txt`, and that it is in the correct folder. Creating the file does not by itself add it to our explicit book table of contents; that is the next step.

## 5. Configure the book

### Step 1: Open the YAML configuration in a code editor

Use VS Code or Notepad for `myst.yml`; use Typora for the `.md` pages. YAML indentation has structural meaning, so a code editor is useful.

We discussed both commands:

```bat
notepad myst.yml
```

```bat
code myst.yml
```

Each opens the configuration in the named editor. `code` requires the VS Code command to be available in the shell. If it is not, open VS Code normally and use **File → Open File**.

### Step 2: Use the final configuration

Replace `myst.yml` with the following, preserving spaces and indentation:

```yaml
version: 1

project:
  id: e363b725-8116-4ab5-8c06-fb3309ead9ea
  title: Jupyter Book Tutorial
  toc:
    - file: index.md
    - file: chapter1.md

site:
  template: book-theme
  title: Jupyter Book Tutorial
  options:
    logo_text: Jupyter Book Tutorial
    hide_footer_links: true
  parts:
    primary_sidebar_footer: _site/primary_sidebar_footer.md
```

Save the file.

| Setting | Purpose in this project |
| --- | --- |
| `version: 1` | Configuration schema version; it does not mean Jupyter Book version 1. |
| `project.id` | Existing project identifier, retained rather than regenerated. |
| `project.title` | Book/project title. |
| `project.toc` | Explicit ordered list of the book pages. |
| First `file` entry | Introductory page, `index.md`. |
| Second `file` entry | Second page, `chapter1.md`. |
| `site.template` | Selects the `book-theme` website template. |
| `site.title` | Website title. |
| `logo_text` | Text used in the theme's branding/header area. |
| `hide_footer_links` | Retains the existing setting to hide footer links. |
| `primary_sidebar_footer` | Uses the existing file as the custom sidebar-footer content. |

The two footer-related settings were preserved from the original project because we wanted to retain its footer customization. **Keep `_site/primary_sidebar_footer.md` in the repository.** We did not inspect or recreate its contents during this session, so this guide does not invent a replacement. On a new project, copying that reference requires also supplying the referenced file.

The old `create_jupyter_book_tutorial.md` remained on disk and in Git. We removed it only from the table of contents. Removing a TOC entry is not the same as deleting a file.

The original configuration also had commented hints such as `# description:` and a suggestion to run `jupyter book init --write-toc`. We did not run that command: we manually listed the two desired pages. Automatic TOC generation could include material we did not intend to show.

### Why preserve the identifier?

This is an existing book, so we kept its project ID. If you create an unrelated book later, let its initialization generate its own identifier rather than copying this project's ID.

## 6. Preview the book locally

### Step 1: Start the server

In Anaconda Prompt, from the repository root, run:

```bat
jupyter book start
```

This builds the content and starts a local website preview. On our first run it fetched the book theme and installed web libraries, so initial startup involved more work than simply opening a file.

The output confirmed that it built:

- `index.md`;
- `chapter1.md`;
- `_site/primary_sidebar_footer.md`;
- two project pages overall.

The footer is a supporting part, not a third chapter.

The terminal then reported that the server started on port 3000 and printed:

```text
http://localhost:3000
```

### Step 2: Open the address in a browser

Open [the local preview](http://localhost:3000). If your terminal prints a different address or port, use that address instead.

Check that:

- the title is `Jupyter Book Tutorial`;
- the sidebar includes the introduction and `My First Chapter`;
- you can switch between the pages;
- the equations render;
- your footer customization is retained.

On a narrow browser window, the sidebar can appear as a sliding menu with a shaded background rather than as a permanently visible column. That is a responsive layout, not a build failure.

For subsequent edits, save in Typora and check the preview. If a configuration change is not reflected, restart the preview server.

### Step 3: Stop the preview when needed

The terminal stays occupied while the server is running; that is expected. To regain the command prompt, press **Ctrl+C**. If Windows asks:

```text
Terminate batch job (Y/N)?
```

type `Y` and press Enter.

Stopping this server ends the localhost preview. It does not delete the book or affect a deployed GitHub Pages site.

## 7. Configure automatic publishing

### Step 1: Generate the workflow

After stopping the preview, run:

```bat
jupyter book init --gh-pages
```

The `--gh-pages` option asks Jupyter Book to create a GitHub Actions publishing workflow. We accepted the defaults:

```text
? What branch would you like to deploy from? main
? What would you like to call the action? deploy.yml
```

At each prompt, pressing Enter accepted the value shown in parentheses.

The command created:

```text
.github/workflows/deploy.yml
```

The `main` answer chooses the source branch used for publishing. `deploy.yml` is the workflow filename; it is not a book page.

We did not hand-write this workflow. Its exact generated contents were not pasted into our conversation, so this guide intentionally documents the generator rather than inventing the YAML. The actual committed workflow is available [in the repository](https://github.com/MichaelBarmann/jupyter-book-tutorial/blob/main/.github/workflows/deploy.yml).

For this already-configured repository, do not regenerate the workflow every time you edit a page. This is a setup step. [Official publishing procedure](https://jupyterbook.org/stable/get-started/publish/)

### A typo we encountered

This command failed:

```bat
jupyter book init -gh-pages
```

It produced `error: unknown option '-gh-pages'`. The correct command has **two hyphens**:

```bat
jupyter book init --gh-pages
```

### Step 2: Make the repository eligible for GitHub Pages

Initially the repository was private, and its Pages settings displayed:

> Upgrade or make this repository public to enable Pages

We chose to make both the repository and published book public for this tutorial.

In GitHub:

1. Open the repository's **Settings → General** page.
2. Scroll to **Danger Zone**.
3. Find **Change repository visibility**.
4. Select **Change visibility → Change to public**.
5. Complete GitHub's confirmation prompts.

Making a repository public exposes its repository contents and history, not only the pages in the book's TOC. Our existing tutorial document therefore remained part of the public repository.

For future projects, decide visibility before publishing. A private source repository and a private website are separate requirements. GitHub Free supports Pages from public repositories; supported paid plans can publish from private repositories, but that alone does not provide a private website. [GitHub Pages availability](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages)

We discussed other hosting arrangements but did not implement them. This tutorial describes the public setup that actually succeeded.

### Step 3: Select GitHub Actions as the Pages source

Open [Settings → Pages](https://github.com/MichaelBarmann/jupyter-book-tutorial/settings/pages).

Under **Build and deployment**, set **Source** to **GitHub Actions**.

If you are on **Settings → General**, you will not see that section. Choose **Pages** from the left settings sidebar, scrolling down if necessary, or use the direct link above.

We used an Actions-based deployment. We did not select a `gh-pages` branch, run `ghp-import`, or manually upload `_build/` to GitHub.

## 8. Commit, push, and check deployment

### Step 1: Review local changes

In Anaconda Prompt, from the repository root:

```bat
git status
```

This reports the branch, known relationship to the remote branch, and changed/untracked files. It does not change them or fetch new remote information.

Our expected changes were:

```text
modified:   index.md
modified:   myst.yml

Untracked files:
    .github/
    chapter1.md
```

`index.md` and `myst.yml` were already tracked and now modified. `chapter1.md` and the workflow were new. Generated `_build/` files did not appear because `.gitignore` excluded them.

### Step 2: Stage the four intended files

```bat
git add index.md myst.yml chapter1.md .github/workflows/deploy.yml
```

`git add` stages the current versions of these files for the next commit. It does not upload them. We named the files explicitly so the commit contained the intended changes.

Git showed a warning about LF being replaced by CRLF for the workflow. These are line-ending conventions. In our case it did not stop the commit or deployment, and no corrective action was needed.

### Step 3: Commit locally

```bat
git commit -m "Create two-page book and configure GitHub Pages"
```

`git commit` records the staged changes in local history. `-m` supplies the descriptive message. Our successful output included:

```text
[main ef1613b] Create two-page book and configure GitHub Pages
4 files changed, 71 insertions(+), 39 deletions(-)
```

The commit ID and change counts will differ in another run.

The command actually typed in our session omitted the space after `-m`:

```bat
git commit -m"Create two-page book and configure GitHub Pages"
```

Git accepted it. The spaced form above is easier to read; both expressed the same message.

### Step 4: Push to GitHub

```bat
git push origin main
```

- `origin` is the remote created when the repository was cloned.
- `main` is the branch being uploaded.
- A successful push updates the remote branch and triggers the configured workflow.

Our output ended with `main -> main`, confirming that the branch was updated.

### Step 5: Watch the workflow

Open the repository's [Actions tab](https://github.com/MichaelBarmann/jupyter-book-tutorial/actions). Open the run associated with the commit message.

| Status | Meaning |
| --- | --- |
| Yellow/pending/running | Wait for it to finish. |
| Green check / Success | The workflow completed. |
| Red cross / Failure | Open the failed job and step to inspect the error. |

Our first deployment succeeded in approximately 35 seconds. That was one observed run, not a guaranteed duration.

There was also a Node.js deprecation warning concerning actions used by the generated workflow. It did not prevent this deployment. The warning concerned the action runtime on GitHub, not proof of a problem with the local Node.js installation. If it becomes relevant later, inspect the exact action named in the current log before changing anything.

### Step 6: Verify the actual website

Open [the published book](https://michaelbarmann.github.io/jupyter-book-tutorial/).

We confirmed that both pages were accessible and that the equations rendered. Always check the website as well as the green workflow result: successful deployment and correct page content are related but distinct checks.

## 9. Add one-click access from GitHub

We added the website URL to the repository's **About** section:

1. Open the [repository's main Code page](https://github.com/MichaelBarmann/jupyter-book-tutorial).
2. Click the **gear icon beside About** on the right.
3. Put this address in the **Website** field:

```text
https://michaelbarmann.github.io/jupyter-book-tutorial/
```

4. Click **Save changes**.

The published book can now be opened with one click from the repository. This edit changes repository metadata; it does not require a local Git commit or pull.

We discussed a README link but did not add one during the session. If you want one later, place this Markdown in a repository `README.md`:

```markdown
[Read the Jupyter Book](https://michaelbarmann.github.io/jupyter-book-tutorial/)
```

A README is GitHub's repository introduction. `index.md` is our book's introduction. They can be separate files with different purposes.

## 10. Return to the project and make updates

### A. The short routine for existing pages

Open Anaconda Prompt and go to the repository:

```bat
cd /d "C:\Users\mbarm\Documents\Documents\01_GradSchool\03_code\github_repos\jupyter-book-tutorial"
git status
```

If the working tree is clean and changes may have been made elsewhere, update before editing:

```bat
git pull --ff-only origin main
```

This is an additional maintenance command, not one needed during the original setup. It fetches remote changes and updates the local branch only if Git can fast-forward without a merge. If it refuses, inspect the state rather than forcing the update. If you have unsaved or uncommitted work, finish saving and handle that work before pulling.

Edit `index.md` or `chapter1.md` in Typora and save. Then:

```bat
git status
git add index.md chapter1.md
git commit -m "Update book pages"
git push origin main
```

Check Actions and refresh the website after the deployment succeeds. The `git add` list should name whichever files you actually changed; the example above is for the original two pages.

### B. Preview before pushing when useful

```bat
jupyter book start
```

Open the printed localhost address and review the book. Save edits in Typora. Stop the server with Ctrl+C when you need that terminal for Git commands, or use a second terminal window in the same repository.

You do not need to run a local preview for every tiny wording change. The publishing workflow builds online when you push.

### C. Add another page

For example, create `chapter2.md` in Typora:

```markdown
# My Second Chapter

These are some additional notes.
```

Add it to the existing `project.toc` list in `myst.yml`:

```yaml
  toc:
    - file: index.md
    - file: chapter1.md
    - file: chapter2.md
```

This is a **fragment** to replace the existing `toc` block, not the entire configuration. Keep the rest of `myst.yml`, including the footer settings.

Preview, then stage and publish:

```bat
git add chapter2.md myst.yml
git commit -m "Add second chapter"
git push origin main
```

Both the new page and the updated TOC must be committed.

### D. Include an image

This is an extension of our workflow; we discussed images but did not add one to the two-page book.

Store the image inside the repository, for example `images/my-figure.png`. Reference it from a root-level page with:

```markdown
![Description of the figure](images/my-figure.png)
```

Use a relative path, not `C:\Users\...`. A path on your Windows computer will not exist on GitHub's build machine or in a visitor's browser. After inserting an image with Typora, inspect its reference in Source Code Mode and ensure the actual image is inside the repository.

Publish the page and image together:

```bat
git add chapter1.md images/my-figure.png
git commit -m "Add figure to first chapter"
git push origin main
```

Relative paths depend on the page location. For a page inside `tutorial/`, an image in a root-level `images/` folder would use `../images/my-figure.png`.

### E. What does not need to be repeated?

Once the setup works, do not repeat these for normal content edits:

- cloning the repository;
- reinstalling Jupyter Book;
- generating `deploy.yml`;
- changing repository visibility;
- selecting the Pages source;
- adding the About website link.

The usual work is saving files, committing, and pushing. Add a TOC entry only when the page structure changes.

## 11. Troubleshooting

### “jupyter is not recognized”

Open **Anaconda Prompt** and run `jupyter book --version`. In our case, it worked there immediately because the installation belonged to the Anaconda environment. Do not reinstall merely because ordinary PowerShell cannot find it.

### “myst is not recognized”

We checked it during diagnosis but used `jupyter book` for the successful setup. A standalone `myst` command is not needed to follow this guide when `jupyter book` works.

### “unknown option '-gh-pages'”

Use two hyphens: `jupyter book init --gh-pages`.

### The heading displays as literal `# ...`

Open Typora's Source Code Mode. Remove a backslash before the heading marker, such as `\#`, and ensure the content is not enclosed in a code fence. Paste raw Markdown into Source Code Mode when replacing a complete document.

### The equation remains literal text

For display math, put opening and closing `$$` on their own lines, or type `$$` then Enter in the formatted editor to create a block. For inline math, enable Inline Math in Typora's Markdown preferences and restart Typora.

Typora's rendering settings do not necessarily control equation numbering or appearance in the built Jupyter Book; check the book preview for the final result.

### “Build and deployment” is missing

First confirm that you are on **Settings → Pages**, not **Settings → General**. If the Pages page says to upgrade or make the repository public, that is a plan/visibility restriction. In our setup we deliberately made the repository public, then selected GitHub Actions.

### A new page is missing from the sidebar

Check the filename, the `project.toc` entry, and its indentation. Check that the file was saved in the repository. For the online book, also confirm that both the page and configuration were committed and pushed.

### The footer configuration stops working

Confirm that `_site/primary_sidebar_footer.md` still exists and is tracked, and that `myst.yml` still references it. We retained both `hide_footer_links: true` and the custom sidebar-footer reference. Theme changes can affect customization, so start by inspecting the build output and current theme behavior rather than deleting unrelated files.

### `localhost:3000` does not load

Check whether `jupyter book start` is still running and whether it printed a different port or an error. The localhost server stops when its process exits. Use the public website URL when you want the deployed copy.

### The website shows old content

Check these in order:

1. Did you save the file in Typora?
2. Does `git status` show an uncommitted change?
3. Did you stage the correct file and commit it?
4. Did `git push` succeed?
5. Did the latest Actions run finish successfully?
6. Are you opening the published URL rather than a different local preview?
7. After a successful deployment, try a browser refresh, or Ctrl+F5.

### A push is rejected because the remote has new work

Someone, another computer, or the GitHub web editor may have added commits. Do not use force-push as a routine fix. Check `git status` and reconcile local and remote changes. A clean branch with no local divergence can usually be updated by `git pull --ff-only origin main`; a refusal needs closer inspection.

### “Nothing to commit”

There may be no saved changes, or the intended file may be outside the repository. Confirm the file path and save it, then inspect `git status` again. Already committed changes may still need pushing.

### LF/CRLF warning

This concerns line endings. The warning we saw on `deploy.yml` was not a failed operation; the commit, push, and deployment all succeeded. Avoid changing Git's global line-ending settings merely to silence this one warning.

### The Actions run fails

Open the failed run, choose the failed job, expand the failed step, and read its error. Copy the specific error when asking for help. Common checks include YAML indentation, exact file/path spelling, Pages source, and whether all referenced files were committed. Windows can conceal filename-case mistakes that become visible during a Linux-based build.

## 12. Use this guide as book pages

This file is an ordinary Markdown tutorial. You can read it in Typora immediately. It contains fenced examples of Markdown, YAML, and commands so that instructional code is displayed instead of executed.

### Option A: Add the whole guide as one page

1. Copy `jupyter-book-setup-guide.md` into your local repository root.
2. Add it to the existing TOC:

```yaml
  toc:
    - file: index.md
    - file: chapter1.md
    - file: jupyter-book-setup-guide.md
```

3. Preview and publish:

```bat
jupyter book start
```

After checking the result and stopping the server:

```bat
git add jupyter-book-setup-guide.md myst.yml
git commit -m "Add detailed Jupyter Book setup guide"
git push origin main
```

This keeps the original two pages and adds this tutorial as a third page. It does not overwrite `create_jupyter_book_tutorial.md`.

### Option B: Split it into shorter pages

Create a `tutorial/` subfolder in the repository and divide the content as follows:

| Suggested file | Content from this guide |
| --- | --- |
| `tutorial/01-overview.md` | Section 1 |
| `tutorial/02-repository.md` | Section 2 |
| `tutorial/03-environment.md` | Section 3 |
| `tutorial/04-writing.md` | Section 4 |
| `tutorial/05-configuration.md` | Section 5 |
| `tutorial/06-preview.md` | Section 6 |
| `tutorial/07-publishing.md` | Sections 7–9 |
| `tutorial/08-updating.md` | Section 10 |
| `tutorial/09-troubleshooting.md` | Sections 11 and 13 |

For each file, promote its main section heading to `#` and reduce subordinate heading levels accordingly. Keep each fenced code example intact. The table of contents at the start of this combined guide uses within-page links; omit it when splitting, or replace those links with links to the new files.

Extend `project.toc` with the new pages in order:

```yaml
  toc:
    - file: index.md
    - file: chapter1.md
    - file: tutorial/01-overview.md
    - file: tutorial/02-repository.md
    - file: tutorial/03-environment.md
    - file: tutorial/04-writing.md
    - file: tutorial/05-configuration.md
    - file: tutorial/06-preview.md
    - file: tutorial/07-publishing.md
    - file: tutorial/08-updating.md
    - file: tutorial/09-troubleshooting.md
```

Again, this replaces only the TOC block. Keep the remaining configuration. Preview and stage the new folder plus the configuration:

```bat
git add tutorial/ myst.yml
git commit -m "Add setup tutorial chapters"
git push origin main
```

Choose either the combined page or the split pages unless you specifically want duplicate versions in the book.

## 13. Command reference and links

### Essential commands

| Command | Meaning | When to use |
| --- | --- | --- |
| `cd "path"` | Change directory in PowerShell. | Navigate to local files. |
| `cd /d "path"` | Change drive and directory in Anaconda Prompt. | Enter the repository. |
| `git clone URL` | Create a local copy connected to a remote repository. | First setup on a computer. |
| `git remote -v` | Show remote fetch/push addresses. | Verify the GitHub connection. |
| `Get-ChildItem -Force` | List entries including hidden ones in PowerShell. | Inspect the folder. |
| `Get-Content filename` | Display a text file in PowerShell. | Inspect source/configuration. |
| `jupyter book --version` | Report the installed Jupyter Book version. | Check the environment. |
| `node --version` | Report the Node.js version. | Diagnose website tooling. |
| `jupyter book start` | Build and serve a local preview. | Check actual book rendering. |
| `jupyter book init --gh-pages` | Generate a Pages publishing workflow. | Initial publishing setup. |
| `git status` | Show branch and local change state. | Before staging or diagnosing. |
| `git add FILES` | Stage selected file versions. | Before committing. |
| `git commit -m "message"` | Record staged changes locally. | Save a version in Git. |
| `git push origin main` | Upload local `main` commits. | Trigger publishing. |
| `git pull --ff-only origin main` | Fetch and fast-forward from the remote, if possible. | Update a clean local checkout. |

### Final file responsibilities

| Path | Commit it? | Reason |
| --- | --- | --- |
| `index.md` | Yes | Introductory source page. |
| `chapter1.md` | Yes | Second source page. |
| `myst.yml` | Yes | Book configuration. |
| `.github/workflows/deploy.yml` | Yes | Automatic build/deploy instructions. |
| `_site/primary_sidebar_footer.md` | Yes | Existing custom footer source. |
| `.gitignore` | Yes | Shared ignore rules. |
| Images referenced by pages | Yes | Required to build and display images. |
| `_build/` | No, in this setup | Generated files recreated by the build. |

### Where to go when returning after a few months

- Edit pages in Typora.
- Open Anaconda Prompt for this working Jupyter Book installation.
- Run commands in the folder containing `myst.yml`.
- Use `jupyter book start` for a local preview.
- Save, stage, commit, and push to publish updates.
- Check Actions when publication is delayed or fails.
- Open the book from the repository's About link.

### Documentation

This guide primarily records our successful interactive setup, including the observed errors and corrections. For changed software versions or interfaces, consult:

- [Jupyter Book](https://jupyterbook.org/)
- [Jupyter Book publishing guide](https://jupyterbook.org/stable/get-started/publish/)
- [MyST documentation](https://mystmd.org/guide)
- [Typora mathematics](https://support.typora.io/Math/)
- [GitHub Pages overview and plan availability](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages)
- [The actual publishing workflow in this repository](https://github.com/MichaelBarmann/jupyter-book-tutorial/blob/main/.github/workflows/deploy.yml)

The successful setup used `myst.yml` and `jupyter book ...` commands. If an older tutorial instead centers on `_config.yml`, `_toc.yml`, or `jupyter-book ...`, check which major version it targets before combining its instructions with this project.
