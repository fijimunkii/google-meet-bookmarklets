# Google Meet Bookmarklets

A small collection of one-click utilities for Google Meet, distributed as a static site.

**Live:** https://fijimunkii.github.io/google-meet-bookmarklets/

## Bookmarklets

| | What it does |
|---|---|
| 📝 **Longer Gemini Notes** | Switches Gemini meeting notes from Standard → Longer in one click. Skips the pencil → settings → menu dance every meeting. |
| 🌓 **Invert Shared Screen** | Inverts colors on the screen-share area. Handy when someone shares a glaring white doc and you want it dark (or vice versa). |
| 🙈 **Toggle Meeting Controls** | Hides (or restores) Meet's bottom controls bar to free up screen real estate when presenting or focused. |

To install: open the live site, show your bookmarks bar (`⌘⇧B` / `Ctrl+Shift+B`), and drag any blue button to it.

## How the Longer Notes one works

The interesting one. Google Meet doesn't expose a way to persist Notes length between meetings, but it does send the change as an `UpdateMeetingSpace` protobuf RPC when you toggle it via the UI.

The bookmarklet:

1. Hooks `window.fetch` and waits for Meet to fire any `MeetingSpaceService` RPC (heartbeats fire constantly during a meeting).
2. Captures the `Authorization` header and decodes the meeting's space ID from `x-goog-meeting-identifier`.
3. Constructs an `UpdateMeetingSpace` request body targeting `call_info.settings.smart_notes_settings.smart_notes_detail_level = 2` (Longer).
4. Fires the request reusing the captured headers.

The reason the bookmarklet sniffs an in-flight request rather than building one from scratch: even with a verifiably-correct `SAPISIDHASH` computed from the page's `SAPISID` cookie, manual `fetch` calls get rejected as "Request unsafe for trusted domain" — likely tied to Chrome-attached headers (`x-client-data`, `x-browser-validation`) that JS can't reproduce. Reusing a captured header sidesteps that entirely.

Once the bookmarklet has captured a valid header, you can call `setNotesDetailLevel(n)` from the DevTools console to experiment with other enum values:

- `1` = Standard
- `2` = Longer
- `3` = 500 (out of range)

Full reverse-engineering notes are in the collapsible section on the live site.

## Layout

```
.
├── index.html    # The site — all three bookmarklets + docs
└── README.md     # This file
```

## Deployment

Hosted on GitHub Pages, served directly from `master`. No build step.

**One-time setup** (per repo):

1. Repo **Settings → Pages**
2. **Source:** "Deploy from a branch"
3. **Branch:** `master` / `(root)` → Save
4. Wait ~30s for the first deploy

After that, every push to `master` auto-publishes `index.html` to the live URL.

## Caveats

- The two simple bookmarklets target Meet-specific class/controller names (`i8wGAe`, `hVZhab`). Google rotates these occasionally; if either stops working, inspect the new identifier and update.
- The Longer Notes bookmarklet depends on Meet's protobuf RPC schema staying stable. If Google changes the field numbers or adds anti-replay protection, the body construction in `index.html` will need to be re-derived from a fresh wire capture.
