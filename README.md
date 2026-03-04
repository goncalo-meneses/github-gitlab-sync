# How to Push to (and Pull from) GitHub and GitLab

GitHub and GitLab are the most widely used Git remote repository hosting platforms in the world. This guide was built for the Open Interactive Textbooks project (OIT), managed by the Open Interactive Learning Materials (OILM) team under the TU Delft Library.

This guide details the step-by-step process for pushing to GitHub and [TU Delft's GitLab domain](https://gitlab.tudelft.nl/) simultaneously, although it will also work for any other GitLab domain. There is then an extended section added for maintainers, building on the original method, since this guide was created for maintaining [OITs](https://books.open.tudelft.nl/home/catalog/category/interactive-textbooks) and these are published on TU Delft's GitLab. The main concern is that someone may accidentally commit directly to the GitLab repository, when it is only supposed to be used as a mirror. In that case, you can add a separate `gitlab` remote and create a new branch on GitHub which, through a Pull Request (PR), can then be approved or denied - in the former case, the change would be merged into main.

*Based on [How to Push to GitHub and GitLab Simultaneously](https://catherinepope.com/posts/how-to-push-to-github-and-gitlab/) by Catherine Pope, extended with maintainer recovery workflows.*

---

## Prerequisites

For this to work, you need:
- An existing local git repository.
- A GitHub repository with a remote configured.
- A GitLab account.
- [SSH keys for authentication with both GitHub and GitLab](https://gist.github.com/marcoandre1/4b0fbca83104e08d3e729a25a0cba4eb).

---

## General Setup

### Step 1 - Check your current remote configuration

```bash
git remote -v
```

You should see `origin` pointing to GitHub:

```
origin  git@github.com:<user>/<repo>.git (fetch)
origin  git@github.com:<user>/<repo>.git (push)
```

### Step 2 - Add GitLab as an additional push destination

```bash
git remote set-url --add --push origin git@github.com:<user>/<repo>.git
git remote set-url --add --push origin git@<gitlab-instance>:<user>/<repo>.git
```

> **Note:** The first command re-adds GitHub as a push URL. This is necessary because `--add` replaces the default push URL with the new one, so GitHub must be explicitly re-added.

### Step 3 - Verify the configuration

```bash
git remote -v
```

Expected output:

```
origin  git@github.com:<user>/<repo>.git (fetch)
origin  git@<gitlab-instance>:<user>/<repo>.git (push)
origin  git@github.com:<user>/<repo>.git (push)
```

You now have one fetch source (GitHub) and two push destinations. Every `git push`, or explicitly `git push origin`, will push to both automatically.

---

## Maintainer Setup (Optional)

Regular contributors only need the setup above. Maintainers should additionally add GitLab as a separate named remote, so they can fetch from it directly when needed - for example, if someone accidentally pushed to GitLab without pushing to GitHub, causing the two remotes to diverge.

### Step 4 - Add GitLab as a separate fetch remote

```bash
git remote add gitlab git@<gitlab-instance>:<user>/<repo>.git
```

Run `git remote -v` to verify. You should now see:

```
gitlab  git@<gitlab-instance>:<user>/<repo>.git (fetch)
gitlab  git@<gitlab-instance>:<user>/<repo>.git (push)
origin  git@github.com:<user>/<repo>.git (fetch)
origin  git@<gitlab-instance>:<user>/<repo>.git (push)
origin  git@github.com:<user>/<repo>.git (push)
```

---

## Maintainer: Recovering from a Diverged GitLab

If GitLab's `main` has diverged from GitHub, follow these steps to safely bring the GitLab changes into a review branch on GitHub.

### Step 1 - Fetch the GitLab state

Start by fetching the changes made on GitLab's side to your local repository.

```bash
git fetch gitlab
```

### Step 2 - Create a review branch from GitLab's main

Create a new branch where the GitLab changes will be stored for review.

```bash
git checkout -b <new-branch> gitlab/main
```

### Step 3 - Push the review branch to GitHub

Push the branch to GitHub so the GitLab changes are visible there and can be reviewed.

```bash
git push origin <new-branch>
```

### Step 4 - Open a Pull Request on GitHub

On GitHub, open a PR from `<new-branch>` into `main`. Review the diff, then decide what to keep, and merge or close accordingly.

### Step 5 - Resync GitLab once resolved

Once `main` on GitHub is in the correct state, force push to overwrite GitLab:

```bash
git push origin main --force
```

> **Note:** If GitLab blocks the force push due to branch protection, go to **Settings -> Repository -> Protected Branches** on GitLab, temporarily unprotect `main`, push, then re-protect it afterwards.

### Step 6 - Clean up the review branch

```bash
git branch -d <new-branch>
git push origin --delete <new-branch>
```