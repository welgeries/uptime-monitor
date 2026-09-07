# uptime-monitor

Emails me when **https://morellinas.com** goes down.

## How it works

`.github/workflows/uptime.yml` runs every 5 minutes on GitHub's runners and
requests the homepage. A check passes only when the response is **HTTP 200 and
the HTML contains the string `Morellina`** — so a server that returns an empty
or broken 200 still counts as down.

A single failed request does not raise an alarm. The job retries **3 times, 20
seconds apart**, and only fails if all three fail. When it does fail, it reports
which of three things went wrong — no response at all, a non-200 status, or a
200 with the wrong body — so the alert says something useful about the cause.

**A failed workflow run is the alert.** GitHub emails the repo owner on
workflow failure by default, so no SMTP setup, API keys, or third-party service
is involved.

## Why this is its own repo

A checker running on the same machine as the site could never report that
machine being down, so the check has to originate elsewhere. GitHub Actions is
free and unlimited on public repos, while on a private repo every run bills a
minimum of one minute — which makes 5-minute checks cost real money. This repo
therefore contains only the monitor: no site source, no credentials.

## Known limits

- GitHub's cron minimum is 5 minutes, and scheduled runs are deprioritized
  under load — real detection lag is often 10–20 minutes.
- While the site stays down, every run fails, so expect a repeated email until
  it recovers.
- There is no "back up" notification; a run that goes green is the all-clear.
- Scheduled workflows are disabled after 60 days of repo inactivity —
  `keepalive.yml` makes a monthly commit to prevent that.

## Changing what's watched

Edit the `env:` block in `.github/workflows/uptime.yml` (`URL`, `EXPECT_TEXT`,
`ATTEMPTS`, `DELAY`). Run it on demand from the **Actions** tab →
*Uptime — morellinas.com* → **Run workflow**.

## Testing the alert

To confirm alert emails still reach you, run a drill: **Actions** →
*Uptime — morellinas.com* → **Run workflow**, and set the `url` input to
something guaranteed to fail (e.g. `https://morellinas.com/does-not-exist`).
The run fails and GitHub emails you, without touching the scheduled checks.
Leave the input blank to check the real site.
