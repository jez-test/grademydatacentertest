# Grade My Data Center: notes for Claude

This repo is a static website. It deploys to Vercel from GitHub. The collaborators editing it usually have no coding background, so work in plain language and keep changes small and safe.

## What is in the repo

- `index.html`: the whole site. About 1.1 MB, 66 lines, and several lines are hundreds of thousands of characters long. It is a **built (bundled) app**, not hand-written HTML. The page text lives inside minified JavaScript strings (a `<script type="module">` block of roughly 115 KB, followed by library code).
- `community-primer.pdf`, `policymaker-primer.pdf`, `lancaster-case-study.pdf`, `loudoun-case-study.pdf`: downloadable PDFs linked from the page. Do not edit these by hand.
- `primer-img/`: photos and maps used by the page, referenced as `primer-img/<name>.jpg`.
- `.gitignore`: ignores `.DS_Store`.

There is no build step, `package.json`, or source folder here. Vercel serves the files as they are (Application Preset "Other", Root Directory `./`).

## How to edit `index.html` safely

1. Never open and retype the whole file. Never paste it into chat. Use `grep -n` or a short Python script to find the exact text, check it matches the expected number of times, and replace only that string.
2. Change visible words by replacing the exact text string. Text is usually inside quotes in the JavaScript, so keep surrounding quotes, commas, and escape characters (`\"`, `\n`, `’`) intact. Do not introduce raw line breaks or unescaped quotes inside a string.
3. Leave these alone unless the user explicitly asks and you have explained the risk:
   - The `<script id="gmdc-primers">` block near the top. It handles page anchors, the "scroll to" buttons, and the PDF download buttons (`data-pdf` and `data-pdf-name` attributes).
   - The long `window.__GMDC_XLSX="..."` base64 string inside that script. It is an embedded spreadsheet. Changing it by hand will corrupt the download.
   - Any minified library code after the app's own text (React and similar).
4. If you add an image, put the file in `primer-img/` with a lowercase, hyphenated name, and reference it as `primer-img/<name>.jpg`. Keep each image under about 500 KB.
5. If you add or rename a PDF, keep it in the repo root, and update both the link and its `data-pdf` and `data-pdf-name` values.
6. After every edit, confirm the file still loads: check that the `<script>` tags are balanced, and look at the preview. If the page goes blank, undo with `git checkout -- index.html` and try a smaller change.

## What needs a rebuild or a specialist

Layout changes, new sections, new interactive features, or changes to the embedded spreadsheet usually mean the original source project must be changed and rebuilt, because this repo holds only the build output. If a request needs that, say so plainly, describe what it would involve, and ask the user how they want to proceed. Do not try to hand-write large new features into the minified code.

PDFs are generated elsewhere. To update one, the new PDF file must be supplied; replace the file with the same name so existing links keep working.

## Git workflow

- Never commit to `main`. Always create a branch named `edit/<short-description>`.
- Before starting, run `git pull --ff-only` on `main` so you begin from the latest version.
- Commit messages: one plain sentence saying what changed on the site, for example "Update contact email in footer".
- Push the branch and open a pull request into `main`. Merging to `main` publishes the live site. Vercel builds a preview URL for each pull request. Share that link with the user to review.
- If the main HTML file has a merge conflict, stop and tell the user. Do not guess a resolution.
- Do not ask for, accept, or store passwords or access tokens in chat. If a push needs sign-in, give the user the command to run in their own Terminal (`gh auth login` the first time).

## Preview

When a collaborator is editing, publish the changed site as a private preview artifact titled "Preview: grademydatacentertest", so they can see each change before it goes into a pull request. The repo files are always the source of truth: edit the repo, then republish the preview from it. See the `edit-site-with-preview` skill if it is installed.

## Style

- Use plain, neutral wording that matches the surrounding text.
- Do not change the site name, section order, or numbers and sources in the case studies unless asked.
- If a requested change involves a fact, number, or quote, ask the user for the source rather than guessing.
