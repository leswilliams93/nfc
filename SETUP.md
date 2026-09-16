# NFC Work Order Prototype — Setup

One file, `index.html`, is the whole thing. No server, no database, no dependencies.
The same page serves every card; the building/floor/room come from the URL, so you
write a different URL to each card and the page adapts.

---

## The URL scheme

```
https://leswilliams93.github.io/nfc/?b=ZACH&f=2&r=210
                                 │      │     └── room
                                 │      └──────── floor
                                 └─────────────── building abbreviation
```

That's 48 characters. It matters: the NFC chips in credit-card-size tags are
usually **NTAG213, which holds only about 132 bytes of URL**. Short parameter
names keep you well under the limit with room to spare.

Optional extras:

| Parameter | Purpose | Example |
|---|---|---|
| `bn=` | Spell out a building that isn't in the lookup list yet | `&bn=Rudder+Tower` |
| `c=`  | Override the big code line on screen | `&c=ZACH-2-210` |

Everything is optional — open the page with no parameters and it demos as ZACH 210.

---

## Part 1 — Put it online ✅ DONE 2026-09-16

**This is already live — skip to Part 2.**

- Live site: **https://leswilliams93.github.io/nfc/**
- Repo: **https://github.com/leswilliams93/nfc** (public, as free Pages requires)
- Verified serving HTTP 200, byte-identical to the local `index.html`

To change the page later, edit `index.html` here and push:

```bash
export PATH="/c/Users/leswilliams/.local/gh/bin:$PATH"
git add index.html && git commit -m "Update form" && git push
```

The live site updates about a minute after the push. The original manual steps are
kept below in case you ever need to rebuild this from scratch.

<details>
<summary>Original setup steps (already completed)</summary>

You need a URL your phone can reach. GitHub Pages is free, permanent, and HTTPS,
so the demo works on cell data and doesn't depend on your laptop being awake.

1. Go to **github.com** and sign up (free) if you don't have an account.
   Whatever username you pick becomes part of your URL, so keep it short.
2. Click the **+** in the top right → **New repository**.
   - Repository name: `nfc`
   - Visibility: **Public** (Pages requires public on the free plan)
   - Leave everything else alone → **Create repository**
3. On the new empty repo page, click **uploading an existing file**.
4. Drag `index.html` from this folder into the browser window.
   Click **Commit changes**.
5. Go to the repo's **Settings** tab → **Pages** in the left sidebar.
   - Under "Build and deployment", set Source = **Deploy from a branch**
   - Branch = **main**, folder = **/ (root)** → **Save**
6. Wait 1–2 minutes, then reload that Settings → Pages screen. It will show:
   **"Your site is live at https://leswilliams93.github.io/nfc/"**

That's your base URL. Test it in your desktop browser first:

```
https://leswilliams93.github.io/nfc/?b=ZACH&f=2&r=210
```

> **Changing the page later:** go to the repo, click `index.html`, click the pencil
> icon, edit, commit. The live site updates in about a minute. Or drag a new
> `index.html` in via *Add file → Upload files* to replace it wholesale.

</details>

> **Don't bother trying a local server as a shortcut.** Running
> `python -m http.server` on the TAMU laptop works in the laptop's own browser but
> is unreachable from a phone: Windows Firewall has Group-Policy Block rules for
> `python.exe` inbound on the Domain profile, all profiles default to BlockInbound,
> and the shell isn't admin. Block rules beat Allow rules in Windows Firewall, so a
> port exception wouldn't help even with admin. This was tested and confirmed.

---

## Part 2 — Check it on the iPhone

Before touching any NFC card, just text or email yourself the URL and open it in
Safari. Confirm:

- Maroon header, location block reads **ZACH 210 / Floor 2 · Room 210**
- The four options are big enough to tap cleanly
- Picking **Other** slides open a description box
- Submitting with nothing selected shows a red "please choose an issue" message
- Submitting with a choice shows the green receipt with a `WO-` reference number

Tap **Share → Add to Home Screen** if you want it to open full-screen without
Safari's address bar for the demo. (Cards will still open in Safari — that's fine.)

---

## Part 3 — Write the NFC card

iPhones have read NFC tags natively since the iPhone 7 / iOS 14. **No app is
needed to read them.** You only need an app to *write* the card once.

**To write:**

1. Install **NFC Tools** by wakdev from the App Store (free version is enough).
2. Open it → **Write** tab → **Add a record** → **URL / URI**.
3. Type the full URL for that room:
   `https://leswilliams93.github.io/nfc/?b=ZACH&f=2&r=210`
4. Tap **OK** → **Write / X records**.
5. Hold the top edge of the phone flat against the card until it confirms.
6. Repeat per room, changing only `b`, `f`, and `r`.

Don't use **Lock tag** while prototyping — locking is permanent and you'll want to
rewrite these as you iterate. Lock only when you go live.

**To read (what your users will do):** wake the phone, hold the top edge of the
phone against the card. A banner slides down from the top; tap it and Safari opens
the form. The screen must be on and the phone unlocked-or-lockscreen — NFC does
not work when the phone is fully powered off or in Low Power Mode edge cases.

> **Where the antenna is:** on the iPhone the NFC coil sits at the very top edge of
> the back, near the camera. Users instinctively tap the middle of the phone and
> nothing happens. That's why the "TAP PHONE HERE" target on your card mockup is
> the right call — keep it.

---

## Part 4 — The QR code on the card

The QR encodes the **exact same URL** as the NFC chip, so anyone with an Android
or an older phone still gets in. Generate one per room at qr-code-generator.com or
any free generator, paste in the room's URL, download as PNG or SVG, and drop it
into your card layout where the mockup shows it.

---

## Adding your buildings

Open `index.html` and find the `CONFIG` block near the top of the `<script>`:

```js
var CONFIG = {
  endpoint: null,
  buildings: {
    ZACH:  "Zachry Engineering Education Complex",
    MSC:   "Memorial Student Center",
    EVANS: "Evans Library"
  }
};
```

Add a line per building — abbreviation on the left, full name on the right. The
abbreviation is what goes in `?b=`. **Verify the three above against your official
building list**; they were seeded as examples. Any abbreviation not in the list
still works, it just displays the abbreviation alone with no full name under it.

---

## What this prototype does and doesn't do

**Does:** captures location from the card, issue type, free-text description,
optional photo, optional contact name. Shows a receipt with a reference number.
Keeps a log of submissions in the phone's local storage — during a demo you can
prove reports were captured by opening Safari's console and running
`JSON.parse(localStorage.woReports)`.

**Doesn't:** create an actual work order. Nothing leaves the phone. The reference
number is generated on the device, not issued by a maintenance system.

**The one hook to make it real:** set `endpoint` in `CONFIG` to a URL, and every
submit will `POST` this JSON to it:

```json
{
  "ref": "WO-260916-4821",
  "building": "ZACH",
  "buildingName": "Zachry Engineering Education Complex",
  "floor": "2",
  "room": "210",
  "code": "ZACH 210",
  "issue": "Too Cold",
  "description": "started this morning",
  "contact": "Les Williams",
  "photo": false,
  "submitted": "2026-09-16T14:42:03.114Z"
}
```

That endpoint could be an AiM/TMA integration, a Power Automate flow, or a Google
Apps Script writing to a Sheet. The page doesn't care.
