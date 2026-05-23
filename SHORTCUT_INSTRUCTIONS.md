# Daily List — iPhone Shortcut (Advanced version)

This is the instruction sheet for building the Shortcut that pushes your iPhone's **Today** reminders into the Daily List's `inbox/` folder on GitHub. The web app picks them up from there.

**What "advanced" means here:**

- If there's nothing new to push, it doesn't push, and you get a small "nothing new" banner instead of a noisy "pushed 0 items"
- If GitHub rejects the push (bad token, wrong repo, offline), you get a banner that *says so* — it won't fail silently
- On success, you get a banner with the count: "Daily List · pushed 3 new items"
- A companion **Personal Automation** runs the Shortcut at a fixed time every morning, so the list is populated before you sit down

You'll build this in two parts: the **Shortcut itself** (the work), then the **Automation** (the schedule that runs it).

---

## Before you start — you'll need three things

1. **A GitHub repo** with an `inbox/` folder in it. The repo can be the same one hosting the Daily List, or any repo you control.
2. **A Personal Access Token** with `Contents: Read and write` permission on that repo. GitHub → top-right profile → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token. Copy it once; you can't see it again.
3. **Three values to paste in below** as you build. Fill them in here so you don't have to scroll:

   - Owner (your GitHub username or org): `_________________`
   - Repo name: `_________________`
   - Branch (usually `main`): `_________________`
   - Token (starts with `github_pat_...`): `_________________`

---

## Part 1 — Build the Shortcut

Open the **Shortcuts** app on your iPhone. Tap the **+** in the top-right to make a new Shortcut. Tap the name at the top and rename it to **Daily List**.

Each numbered step below is one action you add. To add an action, tap the search bar at the bottom (it says "Search for apps and actions"), type the action's name, and tap it. The action will appear in your Shortcut. Configure it as described, then move to the next step.

> **A note on filling in fields:** Where the steps say "tap the blue word *Reminders*" or "tap the blue word *Title*", those blue words are *magic variables* — they're how Shortcuts passes data from one action to the next. Tap them, then in the picker that pops up, pick the variable name in bold.

### 1. Get today's reminders

Search and add: **Find Reminders**

- Tap **Add Filter**: choose **Is Completed** → set to **false** (the toggle on the right)
- Tap **Add Filter** again: choose **Due Date** → **is** → **today**
- Leave Sort By blank, leave Limit off

### 2. Save it as a variable for later

Add: **Set Variable**

- Variable Name: `Reminders`
- The input (the blue chip) should already be **Reminders** (the output of step 1). If not, tap the input field and pick **Reminders**.

### 3. Count how many there are

Add: **Count**

- Set the **Count** dropdown to **Items**
- The input should be the **Reminders** variable. Tap the input field and pick **Reminders** if it isn't already.

### 4. Save the count as a variable

Add: **Set Variable**

- Variable Name: `Count`
- Input: **Count** (the magic variable output by step 3)

### 5. Branch: if there's nothing new, stop

Add: **If**

- Configure it as: **If `Count` is 0**
  - Tap the blue **Input** chip → pick variable **Count**
  - Tap **Condition** → choose **is**
  - In the value field, type `0`

Inside the **If** block (between "If" and "Otherwise"), add:

   - **Show Notification**
     - Title: `Daily List`
     - Body: `Nothing new today`
   - **Stop This Shortcut**

Anything you add *after* the **End If** at the bottom will only run when there *is* at least one reminder. Make sure the next steps go below the **End If**, not inside the If block.

### 6. Build a clean list of items

Add: **Repeat with Each**

- Input: variable **Reminders**

Inside the Repeat block (between "Repeat with Each" and "End Repeat"), add:

   - **Dictionary** action
     - Tap **Add new item** → choose **Text** → key: `text`, value: tap the input field and pick the magic variable **Repeat Item** → then drill into it: tap **Repeat Item** again and choose **Title**
     - Tap **Add new item** → choose **Text** → key: `remindersId`, value: same trick — magic variable **Repeat Item** → **Identifier**
   - **Set Variable**
     - Variable Name: `ItemDict`
     - Input: the **Dictionary** you just made
   - **Add to Variable**
     - Variable Name: `Items` (just type it — Shortcuts will create it the first time)
     - Input: **ItemDict**

After **End Repeat**, the `Items` variable contains all your reminders as a list of dictionaries.

### 7. Build the payload that will become the inbox file

Add: **Dictionary**

- Tap **Add new item** → **Array** → key: `items` → tap the array, then in the empty array tap **Add new item** → **Variable** → pick **Items**
  - *(If Shortcuts doesn't offer "Variable" as a type inside the array, instead set the key `items` to type **Text** and pick the magic variable **Items**. Either works — GitHub's API will accept it.)*
- Tap **Add new item** → **Text** → key: `addedAt` → value: tap the value field, then magic variable picker → **Current Date** → format as **ISO 8601**

Add: **Set Variable**

- Variable Name: `Payload`
- Input: the Dictionary you just made

### 8. Convert the payload to text

Add: **Get Text from Input**

- Input: **Payload**

(This turns the dictionary into a JSON string.)

### 9. Base64-encode that text

Add: **Base64 Encode**

- Input: **Text** (the output of step 8)
- Encode/Decode: **Encode**
- Line Breaks: **None**

Add: **Set Variable**

- Variable Name: `ContentB64`
- Input: **Base64 Encoded** (the output of the encode action)

### 10. Build a filename for the inbox file

Add: **Current Date**

Add: **Format Date**

- Input: **Current Date**
- Date Format: **Custom**
- Format String: `yyyy-MM-dd'T'HHmmss'Z'`
- Time Zone: **GMT**

Add: **Text**

- In the text field, type: `inbox/`, then tap the variable chip and pick **Formatted Date**, then type `.json`
- Result should read: `inbox/2026-05-22T143000Z.json` (or similar)

Add: **Set Variable**

- Variable Name: `Path`
- Input: **Text** (the output of the Text action above)

### 11. Build the URL

Add: **Text**

- Paste this *exactly*, replacing `YOUR_OWNER` and `YOUR_REPO` with the values you wrote down at the top:
  ```
  https://api.github.com/repos/YOUR_OWNER/YOUR_REPO/contents/
  ```
- Then tap the variable chip at the end and pick **Path**
- Final value should read: `https://api.github.com/repos/YOUR_OWNER/YOUR_REPO/contents/inbox/...json`

Add: **Set Variable**

- Variable Name: `URL`
- Input: **Text**

### 12. Send it to GitHub

Add: **Get Contents of URL**

- Tap the URL field → pick variable **URL**
- Tap **Show More** at the bottom of the action card
- Set **Method** to **PUT**
- Under **Headers**, tap **Add new header**:
  - Key: `Authorization`
  - Value: type `token ` (with a space) then paste your Personal Access Token after it
- Tap **Add new header** again:
  - Key: `Accept`
  - Value: `application/vnd.github+json`
- Set **Request Body** to **JSON**
  - Tap **Add new field** → **Text** → key: `message` → value: `Add from Reminders`
  - Tap **Add new field** → **Text** → key: `content` → value: variable **ContentB64**
  - Tap **Add new field** → **Text** → key: `branch` → value: `main` (or whatever branch you wrote at the top)

### 13. Check whether GitHub accepted it

Add: **Get Dictionary from Input**

- Input: **Contents of URL** (the response from step 12)

Add: **Get Dictionary Value**

- Get: **Value**
- Key: `content`
- Dictionary: **Dictionary** (the output of the previous step)

Add: **Set Variable**

- Variable Name: `ContentResponse`
- Input: **Dictionary Value**

GitHub returns a `content` key when it accepts the file and *omits* it (returns `message` instead) when something is wrong.

### 14. Branch: success or failure?

Add: **If**

- Configure as: **If `ContentResponse` has any value**
  - Tap **Input** → variable **ContentResponse**
  - Condition: **has any value**

**Inside the If block** (success):

- **Show Notification**
  - Title: `Daily List`
  - Body: type `Pushed `, then variable **Count**, then ` new items` → final reads "Pushed 3 new items"

**In the Otherwise block** (failure):

- **Get Dictionary Value**
  - Get: **Value**
  - Key: `message`
  - Dictionary: the **Dictionary** from step 13
- **Show Notification**
  - Title: `Daily List sync failed`
  - Body: variable **Dictionary Value** (the error message GitHub sent)

That's the whole Shortcut. Tap **Done** in the top right.

### 15. Test it once

- Go back to the Shortcuts home tab. Tap the **Daily List** tile to run it.
- If you have any incomplete Today reminders, you should see "Daily List · Pushed N new items" within a few seconds.
- If you don't have any, you should see "Daily List · Nothing new today".
- If something's wrong (typo in owner/repo, expired token, no internet), you should see "Daily List sync failed · ..." with the GitHub error.

If the test succeeds, go to GitHub in a browser and look in your repo's `inbox/` folder. You should see a new file named `2026-05-22T...Z.json` (or whatever today's date is). Open it, then look at the **Raw** view to see the base64 content. You don't need to decode it — the web app will.

### 16. Wire it to Siri

- Long-press the **Daily List** tile in Shortcuts → **Details** → **Add to Siri** → record the phrase "Daily List" (or whatever you want to say).
- Now: "Hey Siri, Daily List" runs the Shortcut.

---

## Part 2 — The Personal Automation (runs it every morning)

This is the part that means you don't have to think about running it at all.

1. In Shortcuts, tap the **Automation** tab at the bottom.
2. Tap **+** in the top right → **Create Personal Automation**.
3. Choose **Time of Day**.
4. Set the time to whenever you want the morning sync — I'd suggest **6:30 AM** so the list is ready when you sit down with coffee.
5. Set repeat to **Daily**.
6. Important: at the bottom, **turn OFF "Ask Before Running"** and **turn OFF "Notify When Run"**. This is what makes it truly hands-free.
7. Tap **Next**.
8. Tap **Add Action** → search for **Run Shortcut** → pick the **Daily List** shortcut.
9. Tap **Done**.

Now every morning at 6:30 the Shortcut runs in the background, your Today reminders get pushed to the inbox folder, and you only hear from your phone if there's nothing new (silent), if there are new items (the count banner), or if the sync failed (the error banner). You don't have to lift a finger.

---

## What this Shortcut does NOT do

- **It does not bring completed items back into Reminders.** Checking something off in the Daily List web app does not mark the matching reminder complete. This is one-way by design. If you want them in sync, finish things in both places — or just pick one to be authoritative.
- **It does not dedupe by reminder ID.** It pushes every Today reminder on every run, and the web app's `inbox/` drainer dedupes them by lowercased text. So if you have "Call electrician" in Today on Monday, Tuesday, *and* Wednesday, the web app will only ever show one "Call electrician" entry — but the Shortcut will still push it three times. This is fine; the web app handles it, and it means you can always re-add a reminder to Today and it'll re-surface in the list.

---

## When you want to change something

- **Run at a different time:** Shortcuts → Automation → tap the time-of-day automation → tap the time → adjust.
- **Stop the morning sync temporarily** (vacation, etc.): Shortcuts → Automation → tap the automation → toggle **Enable This Automation** off.
- **Token rotated or expired:** Shortcuts → long-press **Daily List** → Edit → scroll to step 12 (the "Get Contents of URL" action) → tap the **Authorization** header → replace the token after `token `.
- **Different repo or branch:** edit the URL in step 11 and the branch field in step 12.

---

*Generated 2026-05-22. If the Shortcuts UI labels have changed by the time you build this, the action *names* in this doc are the ones to search for — they've been stable across iOS versions for several years.*
