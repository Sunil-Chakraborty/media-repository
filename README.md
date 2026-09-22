# GitHub Media Repository

A centralized media repository for Django applications.

This repository stores the actual media files used by the Django application. The Django application running on PythonAnywhere does **not** need to store copies of these media files.

GitHub acts as the **media source of truth**, while Django provides the **dynamic presentation and management layer**.

---

## 1. Architecture

```text
                         GitHub
                    media-repository
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
          images/       videos/        audio/
             │             │             │
             └─────────────┼─────────────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
           pdf/           text/       documents/
                           │
                           │
                           ▼
                      GitHub API
                           │
                           ▼
                Django / PythonAnywhere
                           │
                     MediaItem table
                           │
                           ▼
                   Django Templates
                           │
                           ▼
                     User Browser
                           │
                           ▼
                    GitHub Raw Files
```

### Principle

```text
GitHub     → stores actual media files
Django     → stores metadata and controls presentation
PythonAnywhere → runs the Django application
Browser    → retrieves media from GitHub
```

Django does **not** need to download and store the media files locally.

---

# 2. Repository Structure

The repository uses separate folders according to media type.

```text
media-repository/
│
├── images/
│   ├── 0001_cover.jpg
│   ├── 0002_flower.jpg
│   └── 0003_rose.jpg
│
├── videos/
│   ├── 0001_introduction.mp4
│   └── 0002_lesson_01.mp4
│
├── audio/
│   ├── 0001_welcome.mp3
│   └── 0002_lesson_01.mp3
│
├── pdf/
│   ├── 0001_notes.pdf
│   └── 0002_exercise.pdf
│
├── text/
│   ├── 0001_introduction.txt
│   └── 0002_summary.txt
│
└── documents/
    └── ...
```

Additional media categories can be added when required.

Examples:

```text
subtitles/
books/
presentations/
spreadsheets/
```

---

# 3. Media Naming Convention

All media files should follow this naming convention:

```text
NNNN_description.extension
```

where:

* `NNNN` = chronological sequence number
* `description` = short human-readable description
* `extension` = actual file extension

Examples:

```text
0001_cover.jpg
0002_flower.jpg
0003_rose.jpg
0004_sunflower.jpg
```

For videos:

```text
0001_introduction.mp4
0002_lesson_01.mp4
0003_lesson_02.mp4
```

For audio:

```text
0001_welcome.mp3
0002_word_01.mp3
0003_word_02.mp3
```

For PDFs:

```text
0001_course_notes.pdf
0002_chapter_01.pdf
0003_exercise.pdf
```

For text:

```text
0001_introduction.txt
0002_chapter_01.txt
0003_summary.txt
```

---

# 4. Why Fixed-Width Numbers?

The sequence number should use the same number of digits for every file.

Recommended:

```text
0001
0002
0003
...
0099
0100
0101
```

This is important because normal alphabetical sorting would otherwise produce incorrect ordering.

### Avoid

```text
1_cover.jpg
2_flower.jpg
10_rose.jpg
11_lotus.jpg
```

Alphabetical sorting can produce:

```text
1_cover.jpg
10_rose.jpg
11_lotus.jpg
2_flower.jpg
```

### Use

```text
0001_cover.jpg
0002_flower.jpg
0010_rose.jpg
0011_lotus.jpg
```

Alphabetical sorting now produces the correct chronological order:

```text
0001_cover.jpg
0002_flower.jpg
0010_rose.jpg
0011_lotus.jpg
```

---

# 5. Sequence Number

The sequence number is primarily used for **presentation order**.

For example:

```text
images/
├── 0001_cover.jpg
├── 0002_diagram.jpg
├── 0003_example.jpg
└── 0004_summary.jpg
```

Django can retrieve these files and sort them by the sequence number.

The sequence is independent within each media folder.

For example, this is valid:

```text
images/0001_cover.jpg
videos/0001_introduction.mp4
audio/0001_welcome.mp3
pdf/0001_notes.pdf
```

The `0001` values belong to different media categories.

---

# 6. GitHub as the Media Source of Truth

The actual files are maintained entirely in this repository.

For example:

```text
images/0001_cover.jpg
```

is physically stored in GitHub.

Django stores metadata about the file rather than another copy of the file.

Conceptually:

```text
GitHub
    │
    │ actual file
    ▼
0001_cover.jpg


Django
    │
    │ metadata
    ▼
MediaItem
    ├── folder = images
    ├── filename = 0001_cover.jpg
    ├── media_type = image
    ├── sequence = 1
    └── is_available = True
```

---

# 7. MEDIA_CDN_URL

The Django application uses a common base URL:

```text
MEDIA_CDN_URL
```

For example:

```text
https://raw.githubusercontent.com/<OWNER>/<REPOSITORY>/main
```

A file such as:

```text
images/0001_cover.jpg
```

therefore becomes:

```text
MEDIA_CDN_URL/images/0001_cover.jpg
```

Similarly:

```text
MEDIA_CDN_URL/videos/0001_introduction.mp4

MEDIA_CDN_URL/audio/0001_welcome.mp3

MEDIA_CDN_URL/pdf/0001_notes.pdf

MEDIA_CDN_URL/text/0001_introduction.txt
```

The Django application can dynamically construct these URLs.

---

# 8. Django MediaItem

The Django application maintains a `MediaItem` model containing metadata.

Example:

```text
MediaItem
--------------------------------
folder
filename
media_type
sequence
title
github_path
github_url
is_available
last_synced
created_at
```

For:

```text
images/0007_rose.jpg
```

Django can store:

```text
folder       = images
filename     = 0007_rose.jpg
media_type   = image
sequence     = 7
github_path  = images/0007_rose.jpg
github_url   = MEDIA_CDN_URL/images/0007_rose.jpg
is_available = True
```

---

# 9. GitHub Synchronization

Django will communicate with GitHub using the GitHub API.

The synchronization process is:

```text
GitHub
   │
   │ GitHub API
   ▼
Django Sync Service
   │
   ▼
MediaItem database
```

The synchronization service examines:

```text
images/
videos/
audio/
pdf/
text/
documents/
```

and discovers the files currently available in GitHub.

---

# 10. Synchronization Rules

When Django performs a synchronization:

### New file

If GitHub contains:

```text
images/0005_sunflower.jpg
```

and Django does not know about it:

```text
GitHub
   ↓
Sync
   ↓
New MediaItem
```

The file becomes available to the Django application.

### Existing file

If the file already exists:

```text
images/0005_sunflower.jpg
```

Django updates its metadata.

### Deleted file

If a file previously known to Django is removed from GitHub:

```text
images/0005_sunflower.jpg
```

the Django record is **not immediately deleted**.

Instead:

```text
is_available = False
```

This preserves the database history.

---

# 11. Synchronization Example

Suppose GitHub initially contains:

```text
images/
├── 0001_cover.jpg
├── 0002_flower.jpg
└── 0003_rose.jpg
```

Django synchronizes the repository:

```text
0001_cover.jpg   → Available
0002_flower.jpg  → Available
0003_rose.jpg    → Available
```

Later:

```text
0002_flower.jpg
```

is deleted from GitHub.

After the next synchronization:

```text
0001_cover.jpg   → Available
0002_flower.jpg  → Unavailable
0003_rose.jpg    → Available
```

The physical source of truth remains GitHub.

---

# 12. Django Does Not Store Media Copies

The intended architecture is:

```text
             ┌─────────────────────┐
             │       GitHub         │
             │                     │
             │ Actual media files  │
             └──────────┬──────────┘
                        │
                        │ URL
                        ▼
             ┌─────────────────────┐
             │       Django        │
             │                     │
             │ Metadata            │
             │ Presentation logic  │
             └──────────┬──────────┘
                        │
                        ▼
                  User Browser
```

This keeps the PythonAnywhere application relatively lightweight.

---

# 13. Media Types

Currently supported categories are:

| Folder       | Media Type | Examples        |
| ------------ | ---------- | --------------- |
| `images/`    | Image      | JPG, PNG, WebP  |
| `videos/`    | Video      | MP4, WebM       |
| `audio/`     | Audio      | MP3, WAV, OGG   |
| `pdf/`       | PDF        | PDF documents   |
| `text/`      | Text       | TXT             |
| `documents/` | Document   | Other documents |

Additional folders can be introduced as the application grows.

---

# 14. Adding a New Media File

The basic workflow is:

### Step 1 — Add the file

For example:

```text
images/0008_jasmine.jpg
```

### Step 2 — Commit and push to GitHub

```text
GitHub
   ↓
images/0008_jasmine.jpg
```

### Step 3 — Synchronize Django

Django runs:

`
