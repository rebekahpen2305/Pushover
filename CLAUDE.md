# Sending Beki a push notification

This repo is a notifier. If you need to alert the repo owner about anything —
a long job finishing, a question you're blocked on, a build going red — you can
send a push notification to her phone from here.

## How to send one

Dispatch the `send-now.yml` workflow on the `main` branch:

- workflow: `send-now.yml`
- ref: `main`
- inputs: `message`, `title`, `delay_seconds` (seconds to wait first; `0` sends
  immediately)

With the GitHub MCP tools that is `actions_run_trigger` / `run_workflow`. The run
takes about ten seconds to start, then sends. Check it landed by reading the run's
job steps: the **Send it** step only passes if Pushover returned `"status":1`, so
a green run means delivered, not merely attempted.

## You do not need any credentials

The Pushover application token and user key live in this repo's Actions secrets
(`PUSHOVER_TOKEN`, `PUSHOVER_USER`). The workflow reads them itself. Never ask
for them, never put them in a file, and never echo them into a log — this repo is
**public**.

## Do not try to call the Pushover API directly

`api.pushover.net` is blocked by the egress policy on Claude Code web sessions;
a direct `curl` fails with a 403 at the proxy before it leaves the container.
This is not something to work around — dispatching the workflow above is the
supported route, because the request then goes out from a GitHub runner instead.

## Calling this from another project

The workflow lives here, so a session working in a different repo needs this one
attached first (`add_repo` with owner `rebekahpen2305`, repo `Pushover`), then
dispatches as above. Nothing needs to be copied into the other project.

## The other workflow

`good-afternoon.yml` sends a daily "Good afternoon" at 2:30pm UK time on a
schedule. It handles the GMT/BST switch by waking at both 13:25 and 14:25 UTC and
only sending during the 2pm London hour. Leave it alone unless asked.
