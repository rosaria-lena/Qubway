# Qubway – rules book
 
This repository holds the source files for the **Qubway** rules, written as a [Quarto](https://quarto.org) book. The book is published automatically with GitHub Pages every time a change reaches the `main` branch.
 
**Live site:** https://rosaria-lena.github.io/Qubway/
 
The rest of this README is the setup and maintenance guide, kept here for future reference.
 
## Contents
 
- [Overview](#overview)
- [Part 1: Create the repository](#part-1-create-the-repository)
- [Part 2: Upload the template files](#part-2-upload-the-template-files)
- [Part 3: Turn on GitHub Pages](#part-3-turn-on-github-pages)
- [Part 4: Add the publishing workflow and get your link](#part-4-add-the-publishing-workflow-and-get-your-link)
- [Part 5: Making changes later](#part-5-making-changes-later)
- [Part 6: Work on your laptop in VS Code and sync with GitHub](#part-6-work-on-your-laptop-in-vs-code-and-sync-with-github)
- [Troubleshooting](#troubleshooting)
- [Glossary](#glossary)
## Overview
 
Your `.qmd` files live in this GitHub *repository* (an online project folder), and a small *workflow* file tells GitHub to turn them into a website and publish it on *GitHub Pages* whenever something changes. You can edit straight in your web browser or on your laptop in VS Code.
 
**How the book is organised**
 
| Part of the book | Folder | Chapters |
| --- | --- | --- |
| Welcome | (top level) | `index.qmd` |
| Part I – Core game: rules for every level | `core/` | `aim.qmd`, `components.qmd`, `setup.qmd`, `turn.qmd`, `quantum-basics.qmd`, `winning.qmd` (chapters 1–6) |
| Part II – Levels | `levels/` | `index.qmd` (intro page with a "levels at a glance" table), then `level-1.qmd` to `level-5.qmd` (chapters 7–11) |
| Appendices | `reference/` | `cards.qmd`, `quick-reference.qmd`, `faq.qmd`, `glossary.qmd` (appendices A–D) |
 
Every level chapter uses the same headings: *What's new*, *Additional components*, *Setup changes*, *New rules*, *New cards*, *Changes to winning* and *Strategy tips*. Each level builds on the one before it, so a level chapter only describes what changes and links back to the core rules instead of repeating them.
 
**Other files**
 
| File | What it is |
| --- | --- |
| `_quarto.yml` | Settings: the book title, author, theme and the chapter list, grouped into parts |
| `custom.scss` | Optional colours for your game |
| `images/` | Pictures used in the book |
| `README.md` | This guide |
| `.github/workflows/publish.yml` | The publishing workflow. Don't delete it |
| `.gitignore` | Stops preview files being uploaded (added in Part 6) |

 

## Part 1: Create the repository
 
A repository (or "repo") is the online folder that holds your book's files.
 
1. Click the **+** icon in the top-right corner of any GitHub page, then **New repository**.
2. Fill in the form:
   - **Repository name:** `Qubway`. This becomes the end of your web address. Use lowercase and hyphens, no spaces.
   - **Description:** optional, for example "Rules for my quantum board game".
   - Choose **Public**. On a free account, GitHub Pages only works with public repositories, which means anyone can see the files.
   - Tick **Add a README file**. This matters: it creates the `main` branch the workflow needs.
   - Leave *.gitignore* and *license* as **None**.
3. Click the green **Create repository** button.
You'll land on your repo's page, at `https://github.com/rosaria-lena/Qubway`. Bookmark it, because this is where you'll come to make changes.
 
## Part 2: Upload the template files
 
1. On your computer, unzip `Qubway-rules-book.zip`. On Windows, right-click it and choose **Extract All**. On a Mac, double-click it. You get a folder called `Qubway-rules-book`.
2. Open that folder so you can see what's inside it: the `core`, `levels`, `reference` and `images` folders, plus `_quarto.yml`, `index.qmd` and the other files.
3. On your repo page on GitHub, click **Add file** (near the top, next to the green **Code** button), then **Upload files**.
4. Select **everything inside the folder except `publish.yml`**, both the files and the four subfolders, and drag it all onto the upload area. Dragging folders works in Chrome, Edge and Firefox; in Safari, upload each folder's contents separately. Don't drag the `Qubway-rules-book` folder itself. Otherwise everything ends up one level too deep and the book won't build.
5. Wait until every file name appears in the list under the upload box. Files inside subfolders appear with their folder name in front, such as `levels/level-1.qmd`.
6. Scroll down to **Commit changes**. In the first box, type a short note such as `Add book template`. Leave **Commit directly to the main branch** selected.
7. Click **Commit changes**.
 
## Part 3: Turn on GitHub Pages
 
Do this **before** adding the workflow in Part 5. The workflow runs as soon as you save it, and it fails if Pages isn't on yet.
 
1. On your repo page, click **Settings**. It's the last tab in the row under the repo name (Code, Issues, Pull requests… Settings). If you don't see it, click the **⋯** at the end of that row.
2. In the left-hand menu, under *Code and automation*, click **Pages**.
3. Under **Build and deployment**, find the **Source** dropdown and change it from *Deploy from a branch* to **GitHub Actions**.
That's all; there's no save button. GitHub may now suggest some ready-made workflows. Ignore them.
 
## Part 4: Add the publishing workflow and get your link
 
The workflow is a recipe that tells GitHub: "whenever the files change, install Quarto, build the book and publish it." It has to live in a folder called `.github/workflows`. That folder name starts with a dot, so it's hidden on most computers, which is why you create it on GitHub instead of uploading it.
 
**Create the file**
 
1. Go back to your repo's main page by clicking the repo name `Qubway` at the top.
2. Click **Add file → Create new file**.
3. In the **Name your file…** box, type exactly `.github/workflows/publish.yml`. Each time you type `/`, GitHub turns the text before it into a folder. That's expected.
4. Paste the text below into the big editing area. Indentation (the spaces at the start of lines) matters, so copy it exactly.
```yaml
name: Publish book
 
on:
  push:
    branches: [main]
  workflow_dispatch:
 
permissions:
  contents: read
  pages: write
  id-token: write
 
concurrency:
  group: pages
  cancel-in-progress: true
 
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Get the files
        uses: actions/checkout@v4
 
      - name: Install Quarto
        uses: quarto-dev/quarto-actions/setup@v2
 
      - name: Build the book
        uses: quarto-dev/quarto-actions/render@v2
 
      - name: Prepare GitHub Pages
        uses: actions/configure-pages@v5
 
      - name: Package the book
        uses: actions/upload-pages-artifact@v3
        with:
          path: _book
 
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Publish to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
 
5. Click the green **Commit changes…** button (top right), then **Commit changes** again in the box that pops up.
**Watch it build**
 
1. Click the **Actions** tab at the top of the repo.
2. You'll see a run with a yellow spinning dot. This is GitHub building your book, and it takes 1–3 minutes.
3. When the dot turns into a **green tick**, the book is published. A **red cross** means something went wrong; see [Troubleshooting](#troubleshooting).
**Get your link**
 
1. Go to **Settings → Pages** again. At the top it now says **Your site is live at** followed by your link, with a **Visit site** button.
2. Your link is: `https://rosaria-lena.github.io/Qubway/`
3. **Optional:** show the link on your repo page. On the repo's main page, click the gear icon next to **About** (right-hand side), tick **Use your GitHub Pages website**, then **Save changes**.

## Part 5: Making changes later
 
Every time you save ("commit") a change on GitHub, the book rebuilds and republishes by itself within 1–3 minutes.
 
**Edit text in a chapter**
 
1. On your repo page, click the file, for example `how-to-play.qmd`.
2. Click the **pencil icon** (*Edit this file*) at the top right of the file view.
3. Make your changes. The file uses Quarto/Markdown: `#` starts a heading, `**bold**` makes bold text, and lines starting with `-` become bullet points.
4. Click **Commit changes…** and then **Commit changes**.
5. Wait for the green tick on the **Actions** tab, then refresh your site. If you don't see the change, force a refresh with Ctrl+F5 on Windows or Cmd+Shift+R on a Mac.
**Add a new chapter**
 
1. Click **Add file → Create new file**, name it, for example, `strategy.qmd`, and start it with a heading line like `# Strategy tips`. Commit it.
2. Open `_quarto.yml`, click the pencil, and add the file name to the chapter list in the position you want, with the same indentation as the others:
```yaml
  chapters:
    - index.qmd
    - setup.qmd
    - how-to-play.qmd
    - quantum-moves.qmd
    - strategy.qmd
    - winning.qmd
    - faq.qmd
```
 
3. Commit the change.
**Add pictures**
 
1. To put pictures in an `images` folder, first use **Add file → Create new file** with the name `images/readme.txt` (any text) and commit it. Then open the `images` folder and use **Add file → Upload files** to add the pictures there.
2. In a chapter, write: `![Board after setup](images/board.jpg)`. The text in square brackets is the caption.
**Change the title, author or look**
 
Edit `_quarto.yml`: change `title:` and `author:`. To try another theme, replace `flatly` with any name from [bootswatch.com](https://bootswatch.com/), such as `sketchy`, `minty` or `lux`. To change colours, edit the hex codes in `custom.scss`.
 
**Undo a mistake**
 
GitHub keeps every version. Open the file and click **History** to see earlier versions. The easiest fix is to open an old version, copy its text, and paste it back with the pencil editor.
 
## Part 6: Work on your laptop in VS Code and sync with GitHub
 
You can keep a copy of the repo on your laptop, edit and preview it in VS Code, and send changes back to GitHub with a couple of clicks. **The golden rule: get the latest version from GitHub before you start, and send your changes back when you finish.**
 
How syncing works, in Git words:
 
- **Clone:** download the repo to your laptop, once.
- **Pull:** bring changes from GitHub down to your laptop.
- **Commit:** save a snapshot of your changes on your laptop, with a note.
- **Push:** send your commits up to GitHub.
- **Sync** is VS Code's button that does pull and push together.
### One-time setup
 
**1. Install the tools.** Restart your laptop afterwards if VS Code doesn't detect them.
 
| Tool | Where to get it | Notes |
| --- | --- | --- |
| VS Code | [code.visualstudio.com](https://code.visualstudio.com/) | The editor |
| Git | [git-scm.com/downloads](https://git-scm.com/downloads) | On Windows, accept the default options. On a Mac, VS Code offers to install it the first time you need it; click **Install** |
| Quarto | [quarto.org/docs/get-started](https://quarto.org/docs/get-started/) | Needed to preview the book on your laptop |
| Quarto extension | In VS Code, click the Extensions icon (four squares) on the left, search **Quarto**, and click **Install** | Adds the Preview button and highlighting for `.qmd` files |
 
**2. Tell Git who you are.** In VS Code, open **Terminal → New Terminal** from the top menu. Type these two lines, pressing Enter after each one, and use your own name and the email of your GitHub account:
 
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```
 
If you'd rather keep your email private, use the "noreply" address shown in GitHub under **Settings → Emails**.
 
**3. Clone the repo (download it).**
 
1. In VS Code, close any open folder (**File → Close Folder**).
2. Click the **Source Control** icon on the left (three dots joined by lines), or press Ctrl+Shift+G (Cmd+Shift+G on a Mac).
3. Click **Clone Repository**, then **Clone from GitHub**.
4. A browser window asks you to sign in to GitHub and authorise VS Code. Allow it, then come back to VS Code.
5. Pick `YOUR-USERNAME/quantum-rules` from the list.
6. Choose where to keep it on your laptop, for example your Documents folder. VS Code creates a `quantum-rules` folder there.
7. When asked, click **Open**. If VS Code asks whether you trust the authors, click **Yes**.
You'll see all your files in the Explorer on the left, including a `.github` folder. Leave that folder alone, because it holds the publishing workflow.
 
**4. Add a `.gitignore` file (important).** Previewing the book on your laptop creates two folders, `_book` and `.quarto`, which must not be uploaded.
 
1. In the Explorer, hover over the folder name at the top and click the **New File** icon.
2. Name it exactly `.gitignore`, with a dot at the start.
3. Paste in these two lines and save with Ctrl+S (Cmd+S on a Mac):
```
/_book/
/.quarto/
```
 
4. Commit and sync it using the daily routine below. Your commit message could be `Add gitignore`.
### Your daily routine
 
1. **Open the project.** In VS Code, use **File → Open Recent → quantum-rules**.
2. **Pull first.** In the Source Control panel, click the **⋯** menu and then **Pull**. You can also click the circular arrows in the blue bar at the bottom of the window. This brings down anything you changed on the GitHub website.
3. **Edit.** Open any `.qmd` file from the Explorer and make your changes. Save with Ctrl+S (Cmd+S on a Mac). Changed files get an **M** (modified) or **U** (new) next to their name.
4. **Preview (optional).** With a `.qmd` file open, click the **Preview** button at the top right of the editor, or press Ctrl+Shift+K (Cmd+Shift+K on a Mac). This preview is only on your laptop.
5. **Commit.** Open the **Source Control** panel. Type a short message in the box at the top, such as `Rewrite measurement rules`, and click **Commit**. If VS Code asks *"There are no staged changes… stage all your changes and commit them directly?"*, click **Yes**, or **Always** so it stops asking.
6. **Push.** Click **Sync Changes**. If it says "This action will pull and push commits", click **OK**. Your changes are now on GitHub.
7. **Check it's live.** On GitHub, the **Actions** tab shows the build running. When the green tick appears, refresh your site.
### Things to know
 
- **Adding images:** drag picture files into an `images` folder in the VS Code Explorer (right-click → **New Folder** to create one). They are committed and synced like any other change.
- **Editing in both places:** you can still edit on the GitHub website. Just **Pull** in VS Code before you start working locally again.
- **If a sync says there's a conflict:** the same lines were changed on GitHub and on your laptop. VS Code opens the file with both versions highlighted. Click **Accept Current Change** (your laptop's version), **Accept Incoming Change** (GitHub's version) or **Accept Both Changes**, then save, commit and sync.
- **Undo changes you haven't committed yet:** in Source Control, right-click the file and choose **Discard Changes**. This permanently throws away those changes.
- **Never commit** the `_book` or `.quarto` folders. If they appear in the Source Control list, check that your `.gitignore` file is saved and spelled correctly.
- **Optional:** in **File → Preferences → Settings**, search `git autofetch` and turn it on. VS Code then tells you when GitHub has newer changes.
## Troubleshooting
 
When a run fails, open the **Actions** tab, click the run with the red cross, then click the step with the red cross to read the error.
 
| What you see | Likely cause | Fix |
| --- | --- | --- |
| Red cross at **Prepare GitHub Pages** or **Publish to GitHub Pages**, mentioning "Pages" or "not found" | Pages isn't set to GitHub Actions | Do Part 4, then on the **Actions** tab click **Publish book → Run workflow → Run workflow** |
| Red cross at **Build the book**, saying `_quarto.yml` isn't found | Files were uploaded inside a `quantum-rules-book` folder | Upload the files again from the repo's main page (dragging the files, not the folder), then delete the extra folder: open each file in it, click ⋯ → **Delete file** |
| Red cross at **Build the book**, with a YAML or "mapping" error | An indentation mistake in `_quarto.yml` or a chapter header | Check that the spaces line up with the lines above. Use spaces, never tabs. Undo via **History** if needed |
| Red cross at **Build the book**, saying a file isn't found | A chapter is listed in `_quarto.yml` but the file name is different | Make the name in the list match the file name exactly, including `.qmd` |
| Nothing appears on the **Actions** tab | The workflow file is in the wrong place | It must be exactly `.github/workflows/publish.yml` |
| Green tick, but the site shows a 404 page | GitHub needs a minute more, or the link is wrong | Wait 2 minutes. Use the exact link shown in **Settings → Pages** |
| Green tick, but old text is still showing | Your browser is showing a saved copy | Force a refresh (Ctrl+F5 or Cmd+Shift+R) |
| A run fails mentioning an action "version" being deprecated | GitHub retired an older version of a step | Update the version number in `publish.yml` (check the Quarto docs) |
 
To re-run the publishing at any time without changing a file: **Actions → Publish book → Run workflow**.
 
## Glossary
 
| Term | Meaning |
| --- | --- |
| Repository (repo) | Your project's online folder on GitHub |
| Commit | Saving a change, with a short note describing it |
| Branch / `main` | A line of versions of your files. You only need `main` |
| Pull / Push | Download changes from GitHub / upload your commits to GitHub |
| Workflow / Action | An automatic job GitHub runs for you, here building and publishing the book |
| GitHub Pages | GitHub's free website hosting |
| `.qmd` | A Quarto file: Markdown text that Quarto turns into a web page |
| `_quarto.yml` | The book's settings file |
 
## Sources
 
- [Quarto: Publishing to GitHub Pages](https://quarto.org/docs/publishing/github-pages)
- [quarto-dev/quarto-actions on GitHub](https://github.com/quarto-dev/quarto-actions)
 
Claude finished the response
