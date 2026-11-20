# Proposal: split the site into per-department repositories

This is a proposal for owner approval, not a decision. It describes when and how to split `nakamotopublius/andersonengineering` if GitHub Pages storage becomes tight, and what GitHub actually does with custom domains and project sites.

## Why this comes up

The live site is a GitHub Pages site published from this repository (`main`, path `/`), at https://andersonengineering.app/. GitHub recommends keeping a repository under 1 GB and strongly under 5 GB, and a published Pages site under 1 GB. A single file may not exceed 100 MB (a hard Git limit, not only a Pages limit). Ordinary Git repository storage is not a metered, billed product the way Git LFS, Packages, or Actions storage are. The binding constraint for this project is the Pages 1 GB guideline, plus the 100 MB per-file cap.

## Current size (measured after the ENGR 505–509 materials and course pages were committed)

As of this writing:

- Working tree (the published site, excluding `.git`): **605 MB**
- Git object store: **about 654 MB**
- Largest published trees: `docs/` 313 MB (mostly slide PDFs for 500, 501, and 503), `media/` 82 MB (Deep Dive podcasts for 500–504), and the raw ENGR 505–509 course folders at **40–44 MB each** (course book, references, transcript, assessments, narration audio, and NotebookLM prompts)

No file in the repository is near 100 MB. The largest current files are slide PDFs around 22 MB and lecture narration files around 3–4 MB.

Headroom to the 1 GB Pages guideline is roughly **400 MB**. A suggested trigger to start a split, rather than waiting until the site is hard against the guideline, is when the working tree reaches **about 800 MB**.

## How big a course is

A useful planning range, from what is already in this repo:

- **Raw course materials, including narration audio:** about **42 MB** per course (the 505–509 folders are 40–44 MB).
- **A finished course on the site:** about **70–90 MB**, once a Deep Dive podcast, seven lecture slide PDFs, the course page, and the docs copies of the book and assessment PDFs are added. ENGR 501 and 503 are in this neighborhood once their slide PDFs in `docs/` and their podcasts in `media/` are counted, even though those files do not live inside `engr-501/` or `engr-503/`.

These are planning figures, not a promise that every later course will land in the same band.

## Per-department estimates at 42–90 MB per course

Using the course counts in the Engineering Library and that 42–90 MB range:

| Department | Courses | At ~42 MB (raw) | At ~90 MB (finished) |
| --- | --- | --- | --- |
| Mechanical | 15 | ~630 MB | ~1.4 GB |
| EECS | 20 | ~840 MB | ~1.8 GB |
| Civil & Environmental | 6 | ~250 MB | ~540 MB |
| Chemical | 15 | ~630 MB | ~1.4 GB |
| AeroAstro | 12 | ~500 MB | ~1.1 GB |
| Materials | 13 | ~550 MB | ~1.2 GB |
| Nuclear | 16 | ~670 MB | ~1.4 GB |
| Biological Engineering | 14 | ~590 MB | ~1.3 GB |
| **Total** | **111** | **~4.7 GB** | **~10 GB** |

A single repo cannot hold a finished multi-department library inside the 1 GB Pages guideline. Even one large department in finished form (Mechanical, EECS, Chemical, Nuclear) would overshoot it. Foundations 500–509 can stay together: ten courses in finished form are on the order of 700 MB–900 MB, plus the site shell.

## What is billed, and what is not

- **Ordinary Git storage** for a normal repository is not charged by the gigabyte. GitHub still asks that repositories stay under 1 GB (and strongly under 5 GB).
- **GitHub Pages** published size should stay under **1 GB**. This is the limit that matters for andersonengineering.app.
- **100 MB per file** is a hard Git limit. Keep slide PDFs, podcasts, and narration well under that, as they already are.
- **Git LFS** is a metered product. GitHub’s included allowance is 10 GiB of LFS storage and 10 GiB of LFS bandwidth per month on GitHub Free, GitHub Pro, and Free-for-organizations plans (250 GiB of each on Team and Enterprise Cloud). Usage above the included amount is billed. The official [GitHub pricing calculator](https://github.com/pricing/calculator) lists **additional LFS storage at $0.07 per GiB-month** and additional LFS data transfer out at $0.0875 per GiB. Official billing docs describe hourly GiB-month metering and point to that calculator rather than restating the unit price. **LFS is a poor fit for this site:** GitHub Pages serves the files in the repository (or a Pages branch), not LFS pointer files, so moving narration into LFS would not reliably publish the audio on the site and would add a metered cost without solving the Pages size problem.
- **GitHub Packages** and **Actions storage** are also metered (with their own included allowances and unit rates). They are not how this site is published.
- **Copilot, Team seats, and similar products** are seat- or subscription-based, not storage-based.
- **Public-repo Actions minutes** and GitHub Pages for a public project site are not drawing on the same storage allowances as LFS. The site should still stay under the Pages size guideline.

## Serving large audio from Google Drive instead of the repo

One way to keep the repo smaller, without a department split, is to host narration and podcast audio on Google Drive (or another host) and link to it from the course page.

Tradeoffs to test before relying on this:

- Drive links are not a good in-page `<audio>` source. Playback and seeking are unreliable compared with files served from the site.
- Drive can show quota, virus-scan, or “preview” interstitials, especially for large or frequently downloaded files.
- Sharing settings, link rotation, and a second system to back up would have to be maintained.

This is worth a short test (one podcast, one lecture-narration file) if a split is being delayed, but it should not be assumed to replace `media/` without that test. **Video should stay on Vimeo**, which is already how Watch lectures and Preview are served.

## How custom-domain paths work for project sites (verified against GitHub docs)

The brief idea that `nakamotopublius.github.io/<repo>/` would automatically become `andersonengineering.app/<repo>/` is **true only in one specific setup**, and it is **not how the site is configured today**.

GitHub’s documentation ([About custom domains and GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages)) says:

> By default, if you set a custom domain for a user site or organization site, that same custom domain will be used for all project sites owned by the same account. … if the custom domain for your user site is `www.octocat.com`, and you have a project site with no custom domain configured that is published from a repository called `octo-project`, the GitHub Pages site for that repository will be available at `www.octocat.com/octo-project`.

A **user site** is the repository named `username.github.io`. A **project site** is any other repository’s Pages site. The automatic `custom-domain/repo-name/` path applies when the custom domain is attached to that user (or organization) site, and the project repository does **not** have its own custom domain.

Today, `andersonengineering.app` is attached to the **project** repository `nakamotopublius/andersonengineering` (this repo has the `CNAME` file). There is no `nakamotopublius.github.io` user-site repo holding the domain. In this setup, a new department repo such as `andersonengineering-mechanical` would publish, by default, at `https://nakamotopublius.github.io/andersonengineering-mechanical/`, **not** automatically at `https://andersonengineering.app/andersonengineering-mechanical/`.

To get department sites under the same custom domain path, one of these would be needed:

1. **User-site path (the documented automatic subpath).** Create a user-site repository named `nakamotopublius.github.io`, move the site shell (and, in the plan below, the Foundations 500–509 listing) into it, attach `andersonengineering.app` to that user site, and **do not** set a custom domain on the department repos. Then a project repo `andersonengineering-mechanical` would be reachable at `https://andersonengineering.app/andersonengineering-mechanical/`. The current project-repo CNAME would have to be removed so it does not override the user-site domain. This repo’s own Pages URL would then also become a subpath unless its content lives on the user site.
2. **Per-department subdomains.** Keep this repo as the apex site and give each department its own custom subdomain, for example `mechanical.andersonengineering.app`, each with its own DNS and Pages custom-domain setting. This does not use the automatic subpath rule; it is extra DNS work, but it leaves the current apex arrangement in place.
3. **Leave department sites on `*.github.io` and link to them.** Simplest mechanically, but the URLs would not all live under andersonengineering.app.

Any split should pick one of these on purpose. Option 1 matches the “same custom domain path” idea, if the owner is willing to move the shell onto a user-site repository.

## Proposed split (for approval)

If the owner approves a split when the main working tree approaches **800 MB**:

1. **Main site** keeps the shell (home page, 404, shared template, crest, `courses.json`, `quiz.html`, `exam.html`, `watch.html`, `view.html`, `refs.html`) and **Foundations ENGR 500–509**, including their docs, media, quizzes, and exams. That set should remain inside the Pages guideline.
2. **One repository per department**, named in a regular pattern such as `andersonengineering-mechanical`, `andersonengineering-eecs`, and so on, each published as a GitHub Pages project site. Course folders, that department’s `docs/` and `media/`, and that department’s course pages live there.
3. **Custom domain.** Prefer option 1 above if department courses should appear at `https://andersonengineering.app/<department-repo>/…`. Confirm DNS and HTTPS on the user site before moving the CNAME off this repo. If option 1 is not approved, use subdomains (option 2) or accept github.io URLs (option 3).
4. **Shared template.** Keep one canonical `index.html` template (the current identical copy used on the home page, 404, and every `engr-50x/index.html`). Copy or sync it into each department repo when the template changes. Course-specific data stays in `courses.json` and the `EX` map, not in forked layout.
5. **Listing.** The main `courses.json` remains the full catalog. Rows for department courses should use absolute URLs (or a repo/base field) so Watch, Book, Quizzes, and the course page can point across repos. Department repos can carry a filtered `courses.json` for their own pages, or can fetch the main catalog if CORS and paths allow. This needs a small, explicit URL convention before the first department repo is created.
6. **Video stays on Vimeo.** Do not move Watch lectures or Preview videos into the repos.

### Migration steps (when the 800 MB trigger is hit, and only after approval)

1. Freeze new heavy binaries on the main repo (slide PDFs, podcasts, narration) for courses that will live in a department repo.
2. Decide the domain option (user-site subpath, subdomain, or github.io) and configure DNS first.
3. Create the first department repo as a sample (one department, or even one course), publish Pages, and verify a course page, a book PDF, a podcast, and the main listing link.
4. Move that department’s course files, docs, and media in an ordinary commit history (copy, then delete from main only after the new site is verified). Never force-push; never delete someone else’s work.
5. Update the main listing to the agreed cross-repo URLs.
6. Repeat department by department. Foundations 500–509 stay on the main site unless the owner later chooses otherwise.

## What this proposal is not

It is not a request to split today. The site is at about **605 MB**, still under the 1 GB Pages guideline, with about 200 MB of planned headroom before the 800 MB trigger. It is also not a plan to use Git LFS for site audio or to treat Google Drive as a drop-in media host without a test.
