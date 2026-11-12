---
name: edit-site-with-preview
description: Let a non-coding collaborator edit a GitHub-hosted static site by chatting with Claude, previewing changes live in a private artifact, then publishing a branch and PR with GitHub Desktop (no command line).
---

# Edit a site with a live preview artifact

Use when a collaborator wants to change a static site that lives in a Git repo (deployed by Vercel from GitHub) and has no coding background. The repo is the source of truth. A private preview artifact stands in for the old edit-the-artifact experience: the collaborator describes a change in plain language, Claude edits the repo files, and the preview artifact updates so they can see it. This skill is not tied to any one person's artifact.

## Ground rules

- The repo files are the only source of truth. Never edit the preview artifact and copy back; always edit the repo file, then republish the preview from it.
- Never commit to `main`. Every change goes on a new branch and through a pull request.
- Collaborators use GitHub Desktop for Git. Never require them to install GitHub CLI or use Terminal.
- Never ask for, accept, or store a GitHub password or Personal Access Token in chat. Claude has no GitHub credentials, so the user does the push in GitHub Desktop.
- The preview artifact is private. Do not share or publish it widely unless the user asks.
- Speak plainly. Describe changes as what the page now says or shows, not as code. Keep replies short.
- Read the repo's `CLAUDE.md` and `README` first if present and follow them (file layout, generated files, do-not-touch areas).

## 0. Check readiness (first time only)

Confirm, fixing what is missing in this order:

1. The Claude desktop app is linked to the user's computer and the repo's folder is connected (Add folder). Check with the device info tool; request folder access if needed.
2. GitHub Desktop (desktop.github.com) is installed and signed in with their GitHub account (File, Options, Accounts). It signs in through the browser and handles branches, commits, pushing, and pull requests with buttons.
3. The repo is cloned. If not, in GitHub Desktop: File, Clone Repository, pick the repo and a folder; then connect that folder with Add folder in the Claude app.
4. The user has Write access to the repo on GitHub (ask the owner to add them under Settings, Collaborators, and accept the emailed invite).

## 1. Start a session

1. Ask the user to confirm GitHub Desktop shows no uncommitted changes on `main` (or help them commit or set aside any they have; never discard them).
2. Have them click Fetch origin and Pull origin in GitHub Desktop so they start from the latest `main`.
3. Have them create a branch: Current Branch, New Branch, named `edit/<short-description>`, e.g. `edit/update-case-study-text`. If the device shell can use Git in the folder, Claude may do these steps itself instead (`git status` clean, `git checkout main && git pull --ff-only`, `git checkout -b edit/<short-description>`).

If the device shell cannot open the folder or Git is unavailable there, Claude can still do everything it needs with the device file tools: list the folder, stage a file, edit it, commit it back.

## 2. Open the live preview

The Artifact tool runs in the cloud and cannot see the user's computer, so stage the site files first.

1. Stage the site files (the main HTML plus any PDFs and image folders) from the repo into the cloud workspace with the device stage tool. Stage assets once per session; stage only changed files afterward.
2. Look for an existing preview with the Artifact list action, matching the title `Preview: <repo name>`. Reuse it if it exists.
3. If none exists, publish the staged main HTML as the page (`file_path`) with the other files in the `files` map at the same relative paths the HTML uses (for example `primer-img/photo.jpg`, `report.pdf`). Title it `Preview: <repo name>`. Keep it private.
4. Tell the user: "Here is your live preview. Tell me what you want changed and I will update it."

## 3. The edit loop

Repeat until the user is happy:

1. The user describes a change in their own words. Ask one short question only if the target is ambiguous (for example, which of two similar headings).
2. Make the edit in place on the user's computer with a targeted command (a short Python read-modify-write or `sed -i`), never by retyping the whole file. Site HTML files can be over 1 MB with very long lines, so locate text with `grep -n` or Python and replace only the exact string; confirm the match count is exactly what you expect before writing. If only the device file tools are available, stage the file, apply the same targeted edit to the staged copy, and commit it back with the file's recorded modified time so newer edits are never overwritten.
3. Do not hand-edit generated or embedded content (PDFs, base64 blobs, minified bundles). If the change needs one, explain what must be regenerated and ask how to proceed.
4. Stage the changed file(s) again and republish the same preview artifact (same URL) so the page updates.
5. Reply with one or two sentences saying what changed on the page and prompting them to check the preview. Offer to undo that change if they dislike it.

## 4. Finish: commit, publish, pull request (all in GitHub Desktop)

When the user says they are done:

1. Show a plain-language summary of what changed. Confirm they want to submit it.
2. Commit: tell them a one-sentence summary to type in GitHub Desktop's Summary box (for example "Update contact email in footer"), then click Commit to the branch. If Claude commits itself and git leaves `.lock` or `tmp_obj_*` files that make later commands fail with "File exists", request delete permission once for the repo folder and remove only those leftovers.
3. Push: click Publish branch (first time) or Push origin. Claude cannot push for them.
4. Pull request: click Create Pull Request in GitHub Desktop (it opens GitHub in the browser). On GitHub confirm the base is `main`, then click Create pull request. If they prefer, give the link `https://github.com/<owner>/<repo>/compare/main...<branch>`.
5. Explain what happens next: Vercel builds a preview URL for the pull request (shown as a comment or check on it); someone with merge rights reviews and merges; merging to `main` publishes the live site.

## 5. Wrap up

Offer to delete the preview artifact if the user is finished, or keep it for the next edit session. If two collaborators are editing at once, remind them to Pull origin first and keep changes small to avoid conflicts. If a pull or merge shows a conflict in the main HTML file, stop and tell the user plainly; do not guess at a resolution.

## Troubleshooting

- Preview looks wrong or missing images: a path in the HTML does not match a staged file. Check that `files` paths match the relative paths in the HTML exactly.
- Preview will not publish (size): the page and each file are limited to 16 MB, and one publish to 64 MB total; stage assets in separate publishes to the same URL if needed.
- GitHub Desktop says it cannot push or access the repo: the user is probably not yet added as a collaborator, or is signed in to the wrong GitHub account. Check File, Options, Accounts and the invite email.
- Push rejected because the branch is behind: someone else changed it; Pull origin, then re-apply or ask the user before any force push.
- Cannot reach the computer: say plainly the session cannot reach their files right now, and ask them to retry shortly; do not edit cloud copies as a substitute.
