# Roxlog — Google Sheet backend

Turns a Google Sheet into the database for your Roxlog install, so the copy
you host on GitHub Pages syncs across every device you train from.

## Setup — five steps, about ten minutes

1. **Make a new Google Sheet.** Name it anything, e.g. `Roxlog`.

2. In that sheet: **Extensions → Apps Script**. Delete whatever is in the
   editor and paste in the whole of `Code.gs` from this folder.

3. Near the top, change `TOKEN` to any random string of your own.
   Save (⌘S / Ctrl+S).

4. **Deploy → New deployment → Web app.**
   - Description: anything
   - **Execute as: Me**
   - **Who has access: Anyone**

   Authorise it when Google asks. It will warn you the app is unverified —
   that is expected for your own script; choose *Advanced → Go to (project)*.
   Copy the **/exec** URL it gives you at the end.

5. Open Roxlog → **More → Google Sheet**. Paste the URL and your token,
   press **Connect & test**, then **Push all up**. Open the sheet and
   your sessions will be sitting in it.

Repeat step 5 on every device you train from — phone, laptop. Nothing else
needs doing again.

## What the sheet looks like

One row per session, on a tab called `sessions`:

| id | date | wk | day | title | mins | km | rpe | hrAvg | hrMax | kcal | result | notes | stations | skipped | updatedAt |
|----|------|----|-----|-------|------|----|----|-------|-------|------|--------|-------|----------|---------|-----------|

`stations` holds the per-station loads as JSON, e.g.
`[{"k":"push","kg":120,"amt":50,"t":"2:14"}]`.

You can edit cells by hand and Roxlog will pick the changes up on the next
pull — just don't change the `id` column, and bump `updatedAt` (any later
ISO timestamp) if you want your edit to win over what's on a device.

## Security, plainly

The Web App has to be readable by "Anyone" because your browser calls it
without signing in to Google. The URL and the token are the only things
protecting it, and **anyone holding both can read and write your training
log**.

Two consequences:

- **Never commit the URL or token to your GitHub repo.** Roxlog stores them
  in your browser's local storage, so the hosted page itself carries no
  secrets. Keep it that way.
- It's a training log, not a bank account. This level of protection is
  proportionate. If it ever leaks, make a new deployment (new URL) and a new
  token.

## API, if you ever want to call it yourself

`GET  <url>?token=YOUR_TOKEN` → `{ok:true, sessions:[...]}`

`POST <url>` with `Content-Type: text/plain;charset=utf-8` and a JSON body:

```json
{ "token": "YOUR_TOKEN", "action": "upsert",  "sessions": [ { "id": "...", ... } ] }
{ "token": "YOUR_TOKEN", "action": "delete",  "ids": ["s123"] }
{ "token": "YOUR_TOKEN", "action": "replace", "sessions": [ ... ] }
```

`text/plain` is deliberate: it keeps the request "simple" so the browser
sends no CORS preflight, which Apps Script cannot answer.

## Troubleshooting

| What you see | What it means |
|---|---|
| "no answer from that URL" | Wrong link (use the `/exec` one, not `/dev`), or access isn't set to Anyone |
| "rejected the token" | `TOKEN` in Code.gs doesn't match what you typed. Change it, **redeploy**, try again |
| Changes don't appear on the other device | Hit **Pull all down**. Roxlog only pulls on load |
| Everything vanished | Your sessions are still in the sheet. **Pull all down** |

Editing `Code.gs` does nothing until you redeploy: **Deploy → Manage
deployments → edit (pencil) → Version: New version → Deploy.** This keeps
the same URL.
