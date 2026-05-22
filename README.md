# The Daily List

A single-page to-do app with the data stored as `data.json` in this repo. The page reads and writes that file via the GitHub API, so the list syncs across devices anywhere you open the page.

## What's in here

- `index.html` — the whole app (HTML + CSS + JS, one file)
- `data.json` — your list (the "database")
- `inbox/` — drop folder for iPhone Shortcut imports (auto-created on first import)

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

Because Apple doesn't let web pages read Reminders, we bridge through the Shortcuts app. The Shortcut grabs your reminders and drops a small JSON file into the `inbox/` folder in this repo. Next time you open the page (or every five minutes while it's open), it merges those items into your list and deletes the inbox file.

### Build the Shortcut

On your iPhone, open the **Shortcuts** app, tap **+** to make a new one, and add these actions in order:

1. **Find Reminders where** — set filters that match what you want imported (e.g., "List is Inbox" and "Is Completed is false"). The narrower the filter, the less chance of accidentally importing your whole life.
2. **Repeat with Each** (input: the Reminders from step 1)
   - inside the loop, add **Get Details of Reminder** → "Title"
   - then **Add to Variable** → name it `Lines`
3. After the loop: **Combine Text** with `Lines`, separator: New Lines. That gives you a plain-text list of titles.
4. **Replace Text** — Find: regex `^(.+)$` (turn on Regular Expression), Replace with: `"$1",`. (This wraps each line in JSON-style quotes.)
5. **Text** action — content: `{ "items": [ ` + the result of step 4 + ` ] }`. The trailing comma is fine — JSON parsers used here tolerate it; if yours doesn't, add a Replace Text step that strips the last `,`.
6. **Base64 Encode** the text.
7. **Get Contents of URL** — this is the actual GitHub API call:
   - URL: `https://api.github.com/repos/YOURNAME/daily-list/contents/inbox/from-iphone-[CurrentDate].json` (use the Current Date variable, formatted as ISO 8601 without colons — colons aren't allowed in filenames)
   - Method: **PUT**
   - Headers:
     - `Authorization`: `Bearer YOUR_TOKEN`
     - `Accept`: `application/vnd.github+json`
   - Request Body: JSON
     - `message`: `import reminders`
     - `content`: the base64 string from step 6
     - `branch`: `main`

Run it. If it returns a 201, the file landed in `inbox/`. Open the page and the items will be in your list within five minutes (or immediately if you trigger a re-sync by refocusing the tab).

### Recommendations

- Make a dedicated Reminders list (e.g., "To Daily List") and have the Shortcut only pull from there — that way you stay in control of what gets imported.
- Pin the Shortcut to your home screen for one-tap import.
- The token in the Shortcut is sensitive. Use a fine-grained token scoped only to this repo, with only Contents: Read/Write.

## How the data file works

`data.json` looks like this:

```json
{
  "items": [
    { "id": 1700000000000, "text": "DESIGN DEV REVIEW", "done": false, "deferUntil": null },
    { "id": 1700000000001, "text": "CALL JANE ABOUT SITE", "done": true,  "deferUntil": null }
  ]
}
```

Fields:
- `id` — a timestamp, used as a unique key
- `text` — the to-do
- `done` — true if checked off
- `deferUntil` — milliseconds-since-epoch; the item hides until that moment. `null` means show now.

You can hand-edit this file in the GitHub web editor any time — the app re-reads it on load.

## Alternatives if a public repo concerns you

If you want the code public but the data private, the easiest move is to put `data.json` in a separate private repo (or a private Gist) and point the Settings panel at that. The app doesn't care where the file lives, as long as the token can read/write it.

## Notes

- The PAT is stored in your browser's localStorage on whichever device you use. It's not sent anywhere except api.github.com. Clear your browser data and you'll need to paste it again.
- If two devices edit at the same time, last write wins. For a single-user list this is fine.
- Fonts come from Typekit (`ikf0hkb`). Without internet the app falls back to system Arial Narrow.
