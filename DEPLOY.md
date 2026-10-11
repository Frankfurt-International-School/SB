# Deploy — G8 PE Profile (v3) to the FIS repo

Target repo: **`Frankfurt-International-School/SB`** (private, currently empty).
Everything below you run once. The app's data stays in the FIS-owned Google Sheet;
nothing here moves student data.

---

## 1 · Get the code into the FIS repo

You have the work as a git bundle: `8profile_v3-combine-profile.bundle` (full history + the
`v3-combine-profile` branch). On your machine:

```bash
# clone the bundle into a working folder
git clone 8profile_v3-combine-profile.bundle fis-pe-profile
cd fis-pe-profile
git checkout v3-combine-profile

# point it at the new FIS repo and push as main
git remote remove origin
git remote add origin https://github.com/Frankfurt-International-School/SB.git
git push -u origin v3-combine-profile:main
```

(If you'd rather keep the old repo's name, push there instead — but the point of the move
is that a repo in the FIS org is the one the student web filter allows.)

Confirm `npm test` is green and the build is current before you rely on it:

```bash
npm run check    # 77 server checks + rebuilds index.html and dist/Code.gs
```

---

## 2 · Turn on GitHub Pages (the student URL)

In the FIS repo → **Settings → Pages** → Source: *Deploy from a branch* → Branch `main`,
folder `/ (root)` → Save. After a minute it gives you a URL, roughly:

> `https://frankfurt-international-school.github.io/SB/`

Note the exact URL — you need it in step 3. **The whole reason for moving to the FIS org
repo is that this URL is allowed on student devices** — confirm with IT that this exact
address is allowlisted for students before you share it.

---

## 3 · Allow the new page to sign students in (the make-or-break)

Google only lets the page sign students in from an **authorised JavaScript origin**. The
sign-in client is the one already in the code (`site.config.json` → `clientId`, public, not
a secret).

- Google Cloud Console → the project that owns that client → **APIs & Services →
  Credentials** → the OAuth 2.0 Client → **Authorised JavaScript origins** → add:
  - `https://frankfurt-international-school.github.io` (the origin only — no path)
- Save. (Leave the existing `scottbain-dot.github.io` origin in place if aa-dash still uses it.)

If sign-in says "origin not allowed", this step is why.

---

## 4 · Deploy the new server (adds the v3 tabs + handlers)

The API is the Apps Script bound to your FIS Google Sheet. v3 adds the Combine, Skills,
Participation, Bands, Goals and Plan tabs and the new save/read actions, so the server
needs redeploying.

1. Open your FIS **Google Sheet** → **Extensions → Apps Script**.
2. Replace the script with the contents of **`dist/Code.gs`** from this repo (paste over the
   old one). Save.
3. Run the menu the Sheet now shows: **PE Profile → 1. Set up tabs** (authorise when asked).
   This creates the new tabs. Your existing Students/Profile/Predictions data is untouched.
4. **Deploy → Manage deployments →** your existing web app → **Edit (pencil) → Version: New
   version → Deploy.** Keep *Execute as Me*, *Who has access: Anyone*. The `/exec` URL does
   **not** change, so `site.config.json`'s `apiUrl` stays as-is.

> If the `/exec` URL ever does change, put the new one in `site.config.json`, run
> `npm run build`, and commit `index.html` — that's the only place the page learns the URL.

---

## 5 · Who can do what (already built in)

- **Students** sign in and see only their own row. They enter their own **Combine fitness**
  numbers, set **goals**, and build a **workout plan**.
- **Teachers** (anyone on the **Teachers** tab, plus the Sheet owner) get the **class
  dashboard** and can record each student's **skill-ladder levels** — this is the "other
  teachers do the skill tests" path. Add the other G8 teachers' FIS emails to the Teachers
  tab.
- Fill the **Students** tab (Email · Name · Class) with the real roster, then share the
  Pages URL.

---

## 6 · Before you share it widely

- Open the Pages URL yourself and sign in with a **student** test account (not just staff):
  own profile shows, a non-FIS account is refused, a wrong account sees "not on the list".
- `PRIVACY.md` is written for the data-protection lead — give it to them and record the
  answers before go-live.
- End of year: **PE Profile → End of year: clear student data** wipes every student-written
  tab (roster kept).

## What changed in v3 (for the commit / your own notes)

- Model reconciled to the final system: **7 skills × 6 levels**; fitness = mile, push-ups,
  sit-to-stand, broad jump, grip, **30 m** sprint, **Illinois agility**, **stork balance**.
- Server: new tabs + student results/goals/plan entry + teacher skill entry + class reads,
  all token-verified and own-row (teacher reads gated to the Teachers tab). 77 checks green.
- Client: populated profile (fitness trends, skill ladders, participation, report bands),
  the Results & Planning flow, and the teacher class dashboard.
- Lesson 1 self-prediction is unchanged.
