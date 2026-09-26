# Comic Forge — deploy & Microsoft Store checklist

## What this is
A fully client-side PWA. No backend, no server cost. Each user's API keys stay in
their own browser (localStorage) and are sent directly from their browser to the
provider they pick — you never see or handle anyone's keys or comic data.

## 1. Host it (free)
GitHub Pages, same as A ONE Removals:
1. Put `index.html`, `manifest.json`, `sw.js`, `icons/` in a repo (or a `docs/` /
   `gh-pages` branch — whatever you already use for AONE-removals-and-uplifts).
2. Settings → Pages → deploy from that branch/folder.
3. **Must be served over HTTPS** — GitHub Pages does this automatically. PWAs
   won't install without it.
4. Visit the URL and confirm the install prompt / "Add to Home Screen" appears.
   If it doesn't, check the browser console for manifest or service-worker errors.

## 2. Package it for the Store
Use [PWABuilder.com](https://www.pwabuilder.com) — free, Microsoft-run:
1. Enter your GitHub Pages URL.
2. It scores your manifest/service worker/icons and flags anything missing.
3. Click "Package for Store" → Windows → download the `.msix` package.

## 3. Partner Center submission
You already have a publisher account (`@TMHP.com`), so skip straight to a new
submission — no new registration fee (Microsoft made individual registration
free in Sept 2025, and you're already past that step anyway):
1. Partner Center → Apps and games → new product → PWA app.
2. Reserve the app name ("Comic Forge" or whatever you want the listing to say).
3. Upload the `.msix` from PWABuilder in the Packages section.
4. Fill in: description, screenshots (take a few of the app running — required
   sizes are shown in the submission form), category (Photo & Video or
   Productivity fits), age rating questionnaire.
5. **Privacy policy** — required since the app *can* handle personal data
   (API keys, user-authored story content), even though it never leaves the
   device. Host a short page (a GitHub Pages page works fine) stating plainly:
   keys and story data are stored only in the user's browser (localStorage),
   nothing is transmitted to you, and generation requests go directly from the
   user's browser to whichever third-party AI provider they configured, subject
   to that provider's own terms.
6. Submit. Certification is usually same-day to a couple of days for PWAs.

## Known limitations to flag to yourself before submitting
- **localStorage has a size cap** (~5-10MB per origin depending on browser).
  Generated art as base64 data URLs will fill this fast across 100+ panels —
  the app warns on write failure, but for a full run, download panels as you
  go rather than relying on the browser to hold everything.
- **CORS**: Pollinations, Hugging Face Inference, and Google's Generative
  Language API all allow direct browser calls today. OpenAI's images endpoint
  has been inconsistent about CORS for browser-origin requests — if you hit
  a CORS error in testing, the fix is a tiny serverless proxy (a single
  Cloudflare Worker on their free tier forwards the request and adds CORS
  headers) rather than moving the whole app off client-side.
- **Google Gemini image gen needs a billing-enabled project** on the *user's*
  key, same issue as before — the app will surface the 429 clearly rather
  than hanging.

## Adding a 5th (or 6th) generation backend later
In `index.html`, everything routes through the `BACKENDS` object — add one
entry (`async (prompt) => dataURL`) and one `<option>` in the two backend
`<select>` elements. No other file needs to change.
