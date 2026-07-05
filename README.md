# superninja-exporter
This toolkit gives you two separate tools that both save your SuperNinja (super.myninja.ai) work to your own PC:
# SuperNinja Chat Exporter (Windows)

A small tool that runs **on your own Windows PC**, using **your own logged-in
SuperNinja session**, to export all your tasks/chats into organized folders on
your computer. It saves each chat as **Markdown + JSON + HTML** by default (HTML keeps the original formatting: bold, code blocks, lists, links, tables, images).

> **Important honesty note:** SuperNinja does not offer an official bulk-export
> feature or a public API for chat history. This tool works by reading the pages
> of the web app while *you* are logged in. It only ever accesses **your own**
> account content. Because it relies on the site's current layout, a future UI
> change on SuperNinja's side may require adjusting the selectors in
> `config.json` (see Troubleshooting). Use it for your own data and in line with
> SuperNinja's Terms of Service.

---

## What you need first

1. **Python 3.9 or newer** installed on Windows.
   - Download: https://www.python.org/downloads/
   - During installation, **check the box "Add Python to PATH"** (very important).

That's the only prerequisite. Everything else is installed automatically.

---

## Step-by-step

> **Why the extra "start Chrome" step?** Google blocks logins in
> automation-controlled browsers ("This browser or app may not be secure").
> To avoid that, we open your **normal** Chrome ourselves (no automation flags)
> and the exporter simply *attaches* to it. Google sees a regular Chrome, so
> sign-in — including "Sign in with Google" — works.

### 1. Set up (one time only)
- Put this whole `superninja-exporter` folder somewhere easy, e.g. your Desktop.
- **Double-click `1_SETUP.bat`.**
  - It creates a private Python environment and installs Playwright.
  - When you see "Setup complete!", close the window.

### 2. Open Chrome and log in
- **Double-click `0_START_CHROME.bat`.**
- A normal Chrome window opens at SuperNinja. **Log in as you always do**
  (Google sign-in works here). Make sure you can see your list of tasks.
- **Leave this Chrome window OPEN.**

### 3. Test with 5 chats first (recommended)
- With that Chrome window still open, **double-click `3_TEST_FIRST_5.bat`**,
  then press a key when prompted.
- The tool attaches to your open Chrome and exports the first 5 tasks into the
  `SuperNinja_Export` folder.
- Open that folder and check the results look right (see "What you get" below).

### 4. Full export
- Once the test looks good, **double-click `2_RUN.bat`** to export everything.
- (Chrome must still be open and logged in. Your login is remembered in the
  `chrome_profile` folder for next time, so you usually won't need to log in
  again.)

---

## What you get

Inside the `superninja-exporter` folder, a new folder called `SuperNinja_Export`:

```
SuperNinja_Export/
├── manifest.json                # index of everything exported
├── My_First_Task_Name/
│   ├── chat.md                  # human-readable Markdown
│   ├── chat.html                # formatted (bold/code/lists/links) - open in a browser
│   ├── chat.json                # structured data (incl. html_blocks)
│   └── chat_full.txt            # raw full text (safety net)
├── Another_Task/
│   ├── chat.md
│   ├── chat.html
│   ├── chat.json
│   └── chat_full.txt
└── ...
```

Each task gets its **own folder**, named after the task title.

---

## Downloading workspace files (separate tool)

There is a **second, separate** tool that downloads every file from each task's
**workspace**, including files in **subdirectories**. It does this exactly the
way you would by hand:

1. Opens the task.
2. Clicks the **folder icon** in the top toolbar → the **Workspace Files** panel
   opens on the right.
3. Clicks the **"..." (3 dots)** button in the panel header (the row with
   *Name / Modified / Size*).
4. Clicks **"Download all"** → SuperNinja zips the whole workspace (with all
   subdirectories) and the browser downloads it.
5. The tool **captures that zip and saves it into the task's folder** as
   `workspace.zip`, then **unzips it** into a `workspace/` subfolder so the
   files are directly browsable (subdirectories preserved). The `.zip` is kept
   too. *(If capture ever fails, the zip still lands in your browser's normal
   Downloads folder — that's fine.)*

It works exactly like the chat exporter — **one to test, one to do all**:

1. Make sure Chrome is open + logged in (`0_START_CHROME.bat`).
2. **Test:** double-click **`TEST_DOWNLOAD_FIRST_5.bat`** (first 5 tasks).
3. **Full run:** double-click **`RUN_DOWNLOAD_ALL.bat`** — it asks which task
   number to **start** from and **end** at (same as `2_RUN.bat`).

Result:

```
SuperNinja_Export/
├── My_First_Task_Name/
│   ├── chat.md / chat.html / chat.json ...   (from the chat exporter)
│   ├── workspace.zip                          (the "Download all" archive)
│   ├── workspace_download.log
│   └── workspace/                             (auto-unzipped for browsing)
│       ├── report.pdf
│       ├── src/
│       │   ├── main.py
│       │   └── utils/helpers.py               (subdirectories preserved)
│       └── data/results.csv
└── ...
```

The workspace files land in the **same task folder** the chat exporter created,
so run the chat export first if you want them side-by-side (either order works).

**If no files download** (the app's file panel may use different markup):
double-click **`DIAGNOSE_WORKSPACE.bat`** — it opens the first task, tries to
open the workspace panel, and writes **`workspace_dom.txt`**. Send that file
back and the selectors can be tuned (just like we did for the jump-to-top
button).

---

## Handling very large chats (memory)

Extremely long threads used to be able to exhaust Chrome's memory (the
"Aw, Snap / reload" page). The exporter now runs in **memory-safe mode** by
default:

- It **harvests the conversation incrementally** as history loads, instead of
  forcing the entire thread into the browser at once. As it scrolls down, the
  app's virtualizer drops the already-captured off-screen messages, so memory
  stays bounded.
- It periodically asks Chrome to **free memory** (this is why
  `0_START_CHROME.bat` now adds `--js-flags=--expose-gc`).
- The console shows a live `heap=NNNMB` reading so you can watch memory.
- If a page **does** crash mid-export, the tool now **reloads and retries** that
  task (up to 3 attempts) instead of silently skipping it.

You can disable memory-safe mode by setting `"memory_safe": false` in
`config.json`, but there's normally no reason to.

---

## Which browser it uses

By default the tool **attaches to your real Google Chrome** rather than
launching a controlled automation browser. `0_START_CHROME.bat` opens Chrome
normally (with a debug port), you log in there, and the exporter connects to it.
This defeats Google's "This browser or app may not be secure" block.

Relevant settings in `config.json`:

| Setting | Meaning |
|---|---|
| `"connect_mode": "cdp"` | Attach to the Chrome you opened with `0_START_CHROME.bat` (recommended, avoids the Google login block). Set to `null` to instead let the script launch its own browser. |
| `"cdp_port": 9222` | The debug port. Must match the one in `0_START_CHROME.bat`. |
| `"browser_channel"` | Only used when `connect_mode` is `null`: `"chrome"`, `"msedge"`, or `null` (bundled Chromium). |
| `"use_persistent_profile"` | Only used when `connect_mode` is `null`. |

- The `chrome_profile/` folder holds your login so you stay signed in between
  runs. Delete it to sign in fresh.
- **Don't have Chrome?** Install it from https://www.google.com/chrome/ .

## Customizing (optional)

Open `config.json` in Notepad to change behavior:

| Setting | What it does |
|---|---|
| `formats` | Any of `"markdown"`, `"json"`, `"text"`. Default: markdown + json. |
| `folder_organization` | `"task"` (one folder per task) or `"date"` (group by export date). |
| `output_dir` | Name of the export folder. |
| `headless_after_login` | `true` to hide the browser during export (faster). |
| `max_scroll_rounds` | Increase if long chats aren't fully captured. |

---

## Command-line options (optional, for advanced users)

Open a Command Prompt in this folder and run:

```
venv\Scripts\activate
python export_chats.py            # full export
python export_chats.py --limit 10          # only first 10 tasks
python export_chats.py --start 6           # start at task 6 (skip first 5)
python export_chats.py --start 6 --end 50  # tasks 6 through 50 only

# NOTE: 2_RUN.bat now ASKS you which task number to start at (press ENTER
# for 1) and which to end at (press ENTER for "all"), so you can resume
# without re-running tasks you already exported.
python export_chats.py --login    # force a fresh login (if session expired)
```

---

## Troubleshooting

**Setup error: "Microsoft Visual C++ 14.0 or greater is required" / "Failed building wheel for greenlet".**
This happens on **Python 3.13+**: older Playwright versions pin an old `greenlet`
that has no prebuilt wheel for 3.13, so pip tries to compile it (which needs C++
build tools you don't have). The updated `1_SETUP.bat` fixes this by installing a
**modern greenlet from a prebuilt wheel first**. If you already hit this error:
  1. Delete the `venv` folder inside `superninja-exporter`.
  2. Double-click `1_SETUP.bat` again (this version installs greenlet the safe way).

If it *still* fails, your Python is likely too new. Install **Python 3.12** from
python.org (tick "Add Python to PATH"), delete `venv`, and re-run `1_SETUP.bat`.
You do **not** need to install Visual C++ Build Tools.

**"No task links were found."**
Your saved login probably expired, or the sidebar layout changed.
- First try: double-click `2_RUN.bat` and, if it doesn't prompt a login,
  run `python export_chats.py --login` from the command line to log in fresh.

**"Couldn't sign you in / This browser or app may not be secure" (Google block).**
This is exactly why the tool now uses the **attach-to-Chrome** method. Make sure
you:
  1. Double-click `0_START_CHROME.bat` FIRST (don't open Chrome the normal way),
  2. Log in inside that specific Chrome window,
  3. Keep it open, then run `3_TEST_FIRST_5.bat` / `2_RUN.bat`.
Because that Chrome is started without automation flags, Google allows sign-in.

**The SuperNinja page opens all white / blank (no login prompt).**
On a brand-new Chrome profile the app sometimes needs one reload to render.
In the Chrome window opened by `0_START_CHROME.bat`:
  - Press **Ctrl + Shift + R** (hard refresh), or Ctrl + R, and wait a few seconds.
  - Do **not** type `/login` — this app signs in from the main URL
    (`https://super.myninja.ai/`); `/login` is a 404.
  - If still blank after two reloads, wait ~15 seconds and refresh once more.
The exporter also auto-reloads a blank page a few times when it attaches, so in
many cases you can just proceed once you can log in.

**"COULD NOT CONNECT TO CHROME" when running the export.**
The exporter couldn't find the Chrome it needs to attach to. Fix:
  - Make sure you ran `0_START_CHROME.bat` and that Chrome window is still open.
  - Only one Chrome can use the debug port at a time. If you had other Chrome
    windows open, close them, then run `0_START_CHROME.bat` again.
  - If your firewall prompts about Chrome/localhost, allow it.

**`0_START_CHROME.bat` says it can't find Chrome.**
Install Chrome from https://www.google.com/chrome/ , or edit the `.bat` to point
at your `chrome.exe` path (see the comment inside the file).

**A chat's messages look incomplete or come out as one big blob.**
The tool couldn't identify individual messages with its default selectors.
- Open `config.json` and adjust the `message_containers` list. To find the
  right value: in your browser, right-click a chat message → **Inspect**, and
  look at the element's `class` or `data-*` attributes. Add a matching CSS
  selector to the top of the `message_containers` list.

**Not all my tasks were exported.**
Increase `max_scroll_rounds` in `config.json` (e.g. to `80`) so the task list
fully loads before discovery.

**Nothing happens / Python not found.**
Reinstall Python and make sure "Add Python to PATH" is checked, then re-run
`1_SETUP.bat`.

**The selectors just don't match the current site.**
Send me a screenshot of the chat page with the browser's Inspect panel open on
a message element, and I'll give you exact selector values to paste into
`config.json`. The scraping logic itself won't need to change — only the
selectors.

---

## How it works (short version)

1. `1_SETUP.bat` builds an isolated Python environment and installs Playwright + Chromium.
2. On first run, a real browser opens; **you** log in; the session cookies are
   saved to `session_state.json` so you don't log in every time.
3. The tool scrolls your task list to load all tasks and collects their links.
4. It opens each task, scrolls to load the full conversation, extracts the
   messages (with best-effort role detection), and writes `chat.md` + `chat.json`
   into a per-task folder.
5. A `manifest.json` index of everything is written at the end.

Your `session_state.json` contains your login cookies — **keep it private** and
don't share it. Delete it anytime to force a fresh login.
