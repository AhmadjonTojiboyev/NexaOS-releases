# Releasing NexaOS

Source lives in the private **NexaOS** repo. Installers are published to the
public **NexaOS-releases** repo, so download links work without a GitHub login
and auto-update can reach them.

---

## Before the first release

### 1. Create the public releases repo

github.com → New repository

- Name: `NexaOS-releases`
- Visibility: **Public**
- Do **not** add a README, .gitignore or licence — it only ever holds installers

Nothing is pushed to it by hand; electron-builder uploads the installer itself.

### 2. Get a token that can write to it

Settings → Developer settings → Personal access tokens (classic) → Generate new
token → scope **`repo`**. Copy the `ghp_…` value.

### 3. Deploy the backend pieces

The app builds fine without these, but staff can't be onboarded:

```bash
supabase login
supabase link --project-ref dbukihrdqbjzohbcngqr
supabase functions deploy admin-invite-member admin-delete-member \
  admin-update-member admin-update-department admin-manage-transaction \
  admin-clear-employee admin-get-screenshot-urls
```

And run `supabase/migrations/20260918000000_support_tickets.sql` in the SQL
Editor, or the Support screen errors out.

---

## Every release

### 1. Push the source

```bash
git add -A
git commit -m "Describe what changed"
git push
```

### 2. Bump the version

`package.json` → `"version"`. electron-updater compares this against the latest
GitHub release, so **a release with an unchanged version will not update
anyone**.

### 3. Build and publish

```powershell
$env:GH_TOKEN = "ghp_..."
npm run release
```

This builds the renderer, packages a Windows NSIS installer, creates a GitHub
release on `NexaOS-releases`, and uploads:

| File | What it's for |
| --- | --- |
| `NexaOS Setup <version>.exe` | the installer people download |
| `latest.yml` | how auto-update finds the new version |
| `*.blockmap` | lets updates download only what changed |

`latest.yml` is not optional — without it, auto-update silently stops working.

### 4. Publish the release

electron-builder creates it as a **draft**. Open
`github.com/AhmadjonTojiboyev/NexaOS-releases/releases`, edit the draft, and
click **Publish release**. Until then nobody can download it.

### 5. Share the link

```
https://github.com/AhmadjonTojiboyev/NexaOS-releases/releases/latest
```

That URL always points at the newest release, so it never needs updating.

---

## Building without publishing

To produce an installer locally and not touch GitHub:

```bash
npm run dist:win
```

Output lands in `dist\`. Useful for testing a build before it goes out.

---

## How updates reach people

Once someone is running NexaOS, `src/main/index.js` checks for updates four
seconds after launch and then every 30 minutes. A newer version downloads in the
background and installs on restart, with a banner offering to restart now.

So this matters: **every release after the first reaches everyone
automatically.** A broken build is not a local mistake — it ships to all 27
people within half an hour. Install it yourself first.

---

## Windows SmartScreen

The installer is unsigned, so Windows shows *"Windows protected your PC"* on
first run. Staff need to click **More info → Run anyway**.

Tell people this in advance — an unexpected security warning on a payroll app
that monitors their screen is exactly the thing that makes someone refuse to
install it.

A code-signing certificate (~$200–400/year, OV or EV) removes the warning. Worth
it before wider rollout, not for this test.
