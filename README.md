# The Daily List

A single-page to-do app with the data stored as `data.json` in this repo. The page reads and writes that file via the GitHub API, so the list syncs across devices anywhere you open the page.

## What's in here

- `index.html` — the whole app (HTML + CSS + JS, one file)
- `data.json` — your list (the "database")
- `inbox/` — drop folder for iPhone Shortcut imports (auto-created on first import)
- `SHORTCUT_INSTRUCTIONS.md` — click-by-click instructions for the iPhone Shortcut
- `SESSION_NOTES.md` — running log of design and behavior decisions

## One-time setup

### 1. Put this on GitHub

```bash
cd "DAILY LIST"
git init
git add .
git commit -m "first commit"
gh repo create daily-list --private --source=. --push
```

Or do it through github.com: create a new private repo called `daily-list`, then drag `index.html`, `data.json`, `.gitignore` into the upload screen.

### 2. Turn on GitHub Pages (so you can open the page from anywhere)

In the repo, go to **Settings → Pages**. Set source to "Deploy from a branch", branch `main`, folder `/ (root)`. Wait a minute, then the page lives at `https://YOURNAME.github.io/daily-list/`.

(If you keep the repo private, GitHub Pages requires a paid plan. If you'd rather keep the repo public but the data private, see "Alternatives" below.)

### 3. Create a Personal Access Token

GitHub → click your avatar → **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**.

- Repository access: **Only select repositories** → pick `daily-list`
- Permissions → Repository permissions → **Contents: Read and write**
- Expiration: whatever you're comfortable with (1 year is fine)

Copy the token. You only see it once.

### 4. Connect the app

Open your page (`https://YOURNAME.github.io/daily-list/` or just double-click `index.html` locally). Click **Settings** in the top right. Fill in:

- Owner: your GitHub username
- Repository: `daily-list`
- Branch: `main`
- Data file path: `data.json`
- Inbox folder: `inbox`
- Personal access token: paste it

Save. You're done. The list now reads from and writes to `data.json` in the repo.

## iPhone Reminders import (via Shortcuts)

Because Apple doesn't let web pages read Reminders, we bridge through the Shortcuts app. The Shortcut grabs your Today reminders and drops a small JSON file into the `inbox/` folder in this repo. The page picks them up from there: on load, on tab focus, and every five minutes while it's open. It dedupes incoming items against the list by their text, so re-pushing the same reminder multiple times is safe — duplicates won't pile up.

The click-by-click build instructions live in **`SHORTCUT_INSTRUCTIONS.md`** in this folder. That's the "advanced" build — silent "nothing new" path so the morning auto-run isn't noisy, explicit error notification if the GitHub call fails, and a Personal Automation that runs the whole thing at 6:30 AM without you touching anything.

### Recommendations

- Use a dedicated Reminders list (e.g., "To Daily List") if you want tight control over what gets imported. The default instructions use the Today smart list, which catches anything due today across all your Reminders lists.
- Add the Shortcut to Siri so "Hey Siri, Daily List" runs it from your phone or HomePod.
- The token in the Shortcut is sensitive. Use a fine-grained token scoped only to this repo, with only Contents: Read/Write.

## How the data file works

`data.json` looks like this:

```json
{
  "items": [
    { "id": 1700000000000, "text": "DESIGN DEV REVIEW", "done": false, "deferUntil": null, "completedAt": null },
    { "id": 1700000000001, "text": "CALL JANE ABOUT SITE", "done": true,  "deferUntil": null, "completedAt": 1700000300000 }
  ]
}
```

Fields:
- `id` — a unique key (created from the timestamp at add-time)
- `text` — the to-do
- `done` — true if checked off
- `deferUntil` — milliseconds-since-epoch; the item hides until that moment. `null` means show now.
- `completedAt` — milliseconds-since-epoch the item was checked off; the app uses this to keep recently-completed items visible inline for 30 seconds before moving them into the Completed archive. `null` for uncompleted items.

You can hand-edit this file in the GitHub web editor any time — the app re-reads it on load.

## Alternatives if a public repo concerns you

If you want the code public but the data private, the easiest move is to put `data.json` in a separate private repo (or a private Gist) and point the Settings panel at that. The app doesn't care where the file lives, as long as the token can read/write it.

## Notes

- The PAT is stored in your browser's localStorage on whichever device you use. It's not sent anywhere except api.github.com. Clear your browser data and you'll need to paste it again.
- If two devices edit at the same time, last write wins. For a single-user list this is fine.
- Fonts come from Typekit (`ikf0hkb`). Without internet the app falls back to system Arial Narrow.
