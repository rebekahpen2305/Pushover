# Pushover playground

A daily **"Good afternoon"** push notification at 2:30pm UK time, sent by a
GitHub Actions cron job. No server, no laptop that needs to be awake — GitHub
runs it.

## Setup

You need two values, and neither one goes in this repo.

### 1. Create a Pushover application

Pushover needs to know *which app* is sending, separately from *who* it's
sending to. Go to **[pushover.net/apps/build](https://pushover.net/apps/build)**,
give it any name (`Good afternoon` works), and create it. You'll land on a page
showing an **API Token/Key** — a 30-character string. That's the first value.

### 2. Find your user key

It's on the front page of [pushover.net](https://pushover.net) once you're
logged in, labelled **Your User Key**. Also 30 characters. That's the second.

### 3. Add both as repository secrets

In this repo: **Settings → Secrets and variables → Actions → New repository
secret**. Add two of them, named exactly:

| Name | Value |
|---|---|
| `PUSHOVER_TOKEN` | the API token from step 1 |
| `PUSHOVER_USER`  | your user key from step 2 |

Secrets are write-only — GitHub won't show them again, and it masks them if
anything tries to print them in a log.

### 4. Try it

**Actions → Good afternoon → Run workflow.** That triggers it by hand, ignoring
the time of day, so you don't have to wait until half two to find out whether it
works. Your phone should buzz within a few seconds.

After that it runs on its own, every day.

## How the timing works

GitHub's scheduler only understands UTC, and the UK doesn't stay on UTC — it's
GMT in winter and BST (UTC+1) in summer. So 2:30pm London is a *different* UTC
time depending on the month.

Rather than editing the cron twice a year, the workflow wakes up at **both**
13:25 and 14:25 UTC, then asks what time it actually is in London and only sends
if it's the 2pm hour. In summer the first one fires and the second skips; in
winter it's the other way round. Nothing to maintain.

### Why it's not exactly 2:30

GitHub runs scheduled jobs on a best-effort basis, and they're often a few
minutes late — occasionally much later, since everyone's cron jobs compete for
runners. Scheduling at :25 rather than :30 buys a little headroom, so it usually
arrives close to the half hour.

The trade-off: if GitHub is more than ~35 minutes behind, the run drifts out of
the 2pm hour and that day is skipped rather than sent at the wrong time. For a
greeting that's the right call. If a guaranteed 2:30 ever matters more than
guaranteed *afternoon*, a scheduler with real cron guarantees is the answer —
GitHub Actions isn't one.

Also worth knowing: GitHub disables scheduled workflows in repos with no
activity for 60 days. Any commit wakes it back up.

## Changing the message

Edit the `title` and `message` lines in
[`.github/workflows/good-afternoon.yml`](.github/workflows/good-afternoon.yml).
To change the time, adjust both `cron` lines *and* the hour the check compares
against — they have to agree, or it'll skip every day.

Pushover's API takes a few other options too — `priority`, `sound`, `url` —
documented at [pushover.net/api](https://pushover.net/api). They're just more
`--form-string` lines.
