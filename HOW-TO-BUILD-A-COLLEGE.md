# How to build an Anderson college site

This guide uses andersonengineering.app as the worked example. It records what was done between November 16 and November 20, 2026, and gives the steps to repeat for another college that sits alongside andersonacademy.app and andersonmedicine.app.

## 1. The pieces

| Piece | Engineering example | Role |
|---|---|---|
| Domain | andersonengineering.app, registered at Name.com | The public address |
| Source repo | [nakamotopublius/andersonengineering](https://github.com/nakamotopublius/andersonengineering) | Holds the site, served by GitHub Pages from `main` at the repo root |
| Course library sheet | Engineering Library Google Sheet | The master list of planned courses and the open sources each one draws on |
| Course build folder | `/workspace/anderson-academy/<CODE>/` (for example `ENGR-501-B/`) | Transcript, narration, whiteboard video, captions, assessments and NotebookLM files |
| Videos | Vimeo, through the official API and `vimeo_upload.py` | The lecture video, a short preview, and an audio-only podcast |
| Build bot | CourseBuilder | Runs the sheet, builds the courses and maintains the site |

## 2. Domain and DNS (one time per college)

Every `.app` domain loads only over HTTPS. Until a certificate is in place the site shows a privacy error.

1. Register the domain. Engineering was registered at Name.com on November 20, 2026.
2. In Name.com, open Manage DNS Records and delete any default parking records (A, ALIAS or CNAME) on the bare domain or on `www`.
3. Add four A records with the host left blank (the bare domain), all pointing to GitHub Pages: `185.199.108.153`, `185.199.109.153`, `185.199.110.153` and `185.199.111.153`. Optional AAAA records for IPv6: `2606:50c0:8000::153`, `2606:50c0:8001::153`, `2606:50c0:8002::153` and `2606:50c0:8003::153`.
4. Add one CNAME record with host `www` pointing to `nakamotopublius.github.io`. Leave the TTL at the default.
5. In the repo, add a file named `CNAME` that contains only the domain (`andersonengineering.app`) and push it to `main`. Set the GitHub Pages custom domain to the same name (in the Pages settings or with the Pages API). GitHub requests a Let's Encrypt certificate once the apex A records resolve. Do not turn on Enforce HTTPS until the certificate state is approved. For Engineering the certificate was issued within minutes and is valid until February 18, 2027; GitHub renews it automatically.
6. If a browser still shows a privacy error after the certificate exists, flush local caches (on a Mac, `sudo dscacheutil -flushcache` then `sudo killall -HUP mDNSResponder`; in Chrome, clear the host cache and socket pools under `chrome://net-internals`), then quit and reopen the browser.
7. The first certificate often covers only the apex. The `www` address needs to be listed on that certificate as well. If GitHub does not add it on its own, remove the custom domain in the Pages settings and add it again.

To check whether the records are live, query a public resolver, for example `curl -s "https://cloudflare-dns.com/dns-query?name=andersonengineering.app&type=A" -H "accept: application/dns-json"`.

## 3. The course library sheet

Each college starts from a Google Sheet that lists the courses it will offer. The Engineering Library sheet has these tabs, in order: Foundations (GIRs); General Engineering Degree Proposal; Track - Mechanical, Track - EECS, Track - Civil & Environmental, Track - Chemical, Track - AeroAstro, Track - Materials, Track - Nuclear and Track - Biological Engineering; and README, always last.

The track tabs share one header row: Order, Course number, Title, Year/term typical, OCW URL, Lecture notes, Problem sets, Exams with solutions, Video lectures, Open-access textbook, MITx/OLL self-paced, MITx/OLL URL and Notes. The Foundations tab adds a final column, Anderson Academy code, which assigns each GIR its code (ENGR 501 B and up). Each tab is also exported as a CSV (for example `foundations-girs.csv` and `track-mechanical.csv`).

For a new college, copy that structure: a foundations tab, one tab per track, a README tab last, and a code column for every planned course. A built course can exist before it is added to the sheet or to the site; see section 6.

## 4. How a course is built

Every course is a Brief course: a five-minute overview (Lecture 0) plus six lectures of about ten minutes, about an hour in all. Its wording is original, and its backbone is open material, credited on the course page and in the Course Book. For Engineering that material is MIT OpenCourseWare, under its Creative Commons BY-NC-SA licence where that licence applies.

Build one sample course and get the owner's approval before building any others, and again before introducing a new format.

Work in the course folder, `/workspace/anderson-academy/<CODE>/`. The build produces:

1. A numbered Course Book (the transcript) and, on the site, `docs/<CODE>-book.pdf` (for example `docs/ENGR-501-B-book.pdf`).
2. Narration with the voice `en-US-ChristopherNeural` at -5%.
3. A narrated whiteboard video at 720p and 10 frames per second, with matching `.srt` and `.vtt` captions. The preview is Lecture 0 plus a 3.2-second end card at 1280×720.
4. A quizzes PDF, a final exam PDF and a Tripos paper PDF, copied to the site as `docs/<CODE>-quizzes.pdf`, `-exam.pdf` and `-tripos.pdf`.
5. When interactive assessments are approved for that course, `quizzes/<SLUG>.json` and `exams/<SLUG>.json`, where the slug has no space or B (for example `ENGR501`), plus matching entries in `quizzes/index.json` and `exams/index.json`.
6. Vimeo uploads, made with `/workspace/course-videos/pipeline/vimeo_upload.py`. Put the token in the environment as `VIMEO_ACCESS_TOKEN` and never print it. Mark a chapter for every lecture, starting at Lecture 0. Add English captions. Set each upload to Public with the content rating "All audiences". The description must include the course name, Anderson branding and the course URL (`https://andersonengineering.app/engr-501/` and so on). Titles:
   - Lecture video and preview: `<CODE> B - <Name> Video (Anderson Engineering)`
   - Audio-only version: `<CODE> B - <Name> Podcast (Anderson Engineering)`
7. A NotebookLM notebook whose only source is the transcript PDF. A Vimeo link does not work as a source (it imports Vimeo's bot-check page, not the video). Always choose "generate later", never "generate now". An account can have at most six "generate later" items scheduled at once.
8. On the site, a pretty URL at `<slug>/`, where the slug is the code in lowercase with a hyphen and no B (for example `engr-501/` for ENGR 501 B). That folder holds a copy of the same `index.html` used on the home page; the course list itself comes from `courses.json`.

As of November 20, 2026, two Engineering courses are on the live site: [ENGR 501 B, Calculus One](https://andersonengineering.app/engr-501/) and [ENGR 503 B, Physics One](https://andersonengineering.app/engr-503/). ENGR 500 B is already built (see section 6) but is not yet listed on the site.

## 5. How courses appear on the site

The home page reads one file, `courses.json`, and draws the course list from it. Each course is one entry with these fields, in order:

```json
[501, "ENGR 501 B", "Calculus One", "vimeo:1248551557",
 "docs/ENGR-501-B-book.pdf", "docs/ENGR-501-B-quizzes.pdf",
 "docs/ENGR-501-B-exam.pdf", "docs/ENGR-501-B-tripos.pdf",
 "Foundations", "1248551989"]
```

| Position | Meaning |
|---|---|
| 1 | Sort number |
| 2 | Course code |
| 3 | Title |
| 4 | Lecture video, written as `vimeo:<id>` (shows the Watch lectures button) |
| 5 to 8 | Course Book, quizzes, final exam and Tripos paper PDFs |
| 9 | Collection heading (for example Foundations) |
| 10 | Vimeo id of the preview (shows the Preview button) |

The site lists only courses that have an entry in `courses.json`. Other files: `watch.html` plays a Vimeo video; `view.html` shows a PDF; `quiz.html` and `exam.html` run the interactive quizzes and final exam; `404.html` is a copy of the same app for unknown addresses; `crest.png` is the college crest.

## 6. Adding a built course that is not yet on the site (ENGR 500 B)

ENGR 500 B, How to Use the Engineering Courses, already exists. It is not a GIR, so it does not appear in the Foundations tab (those codes start at ENGR 501 B). It has a Drive folder ([1yTEl05sqHjV1qaOEmJvghxp3N5EDFEP2](https://drive.google.com/drive/folders/1yTEl05sqHjV1qaOEmJvghxp3N5EDFEP2)), a full narrated course on Vimeo at [vimeo.com/1248517092](https://vimeo.com/1248517092), and a preview at [vimeo.com/1248517199](https://vimeo.com/1248517199). It is not yet in `courses.json` and therefore does not appear on andersonengineering.app.

To put a course like this on the site:

1. Confirm the materials and Vimeo ids.
2. Add its row to the sheet if it should be tracked there.
3. Add its entry to `courses.json`. For ENGR 500 B, use 500 as the sort number so it lists first, and add a folder `engr-500/` containing a copy of `index.html`.
4. If interactive quizzes or an exam are ready and approved, add `ENGR500` to `quizzes/index.json` and `exams/index.json` and add the JSON files.
5. Pull, commit and push to `main`. GitHub Pages publishes the change within a minute or two.

## 7. Quizzes, final exam and Tripos

The question format in both `quizzes/<SLUG>.json` and `exams/<SLUG>.json` is the same: `q`, an `options` array of four answers, `answer` (the 0-based index of the correct option) and `explain` (which cites the transcript paragraph, for example "Paragraph 1.3", and says why the other options are wrong).

- **Interactive quizzes.** `quizzes/<SLUG>.json` has a `title` and a `quizzes` array. For the Brief courses so far there is one quiz per lecture for Lectures 1 through 6 (not Lecture 0), five questions each. The Interactive Quizzes button appears only when the course slug is listed in `quizzes/index.json`. After each choice the explanation is shown at once.
- **Interactive final exam.** `exams/<SLUG>.json` has a `title` and a `questions` array. The Engineering sample is 50 questions. Answers can be changed until submit; unanswered questions get a warning; a 60-minute reference timer runs. After submit the page shows the score, the explanations, and a missed-only view. The Interactive Final Exam button appears only when the slug is listed in `exams/index.json`.
- **Scores** are kept in the visitor's browser under the key `aa_scores`. They are not sent to a server. The course panel shows how many quizzes were taken and the best total.
- **Tripos exam.** The course panel gives a ready-made prompt that names the course and its live page and transcript URLs. The student downloads the Course Book, gives it and the prompt to an AI of their choice, and gets a one-question-at-a-time oral exam in the Cambridge Tripos style. A printable Tripos paper PDF is also provided.

ENGR 501 B has interactive quizzes and an interactive exam. ENGR 503 B currently has the PDF versions only.

## 8. Checklist for a new college

1. Pick the name and domain (for example `anderson<field>.app`) and register it.
2. Create a public repo under `nakamotopublius` and copy the site files from an existing Anderson site: `index.html`, `watch.html`, `view.html`, `quiz.html`, `exam.html`, `404.html`, an empty `courses.json`, and empty `quizzes/index.json` and `exams/index.json`. Change the title, crest and accent colour.
3. Add the `CNAME` file, turn on GitHub Pages from `main` at the root, and set the DNS records as in section 2. Turn on Enforce HTTPS only after the certificate is approved, and confirm that `www` is on the certificate.
4. Build the library sheet as in section 3 and assign course codes.
5. Make one sample course and get approval before building the rest, and before any new format.
6. For each course, build the materials and videos (section 4), add its `courses.json` entry and slug folder, and push.
7. Write a README in the repo that names the college, its address, its source licences and its course address pattern.

Rules that apply to every college:

- Never describe any course or material as free.
- Don't use "pay it forward" wording.
- Anderson lectures run about ten minutes, with a five-minute overview.
- Always pull before pushing, and never force-push or delete another person's work.
- Never print credentials or tokens, including `VIMEO_ACCESS_TOKEN`.
