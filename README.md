# OpenAPI-page

A fixed address for a playground that does not have one.

The [NVIDIA Inference Playground](https://github.com/ByungKwanLee/OpenAPI) is
shared over a `gradio.live` tunnel, and Gradio asks that service for a random
subdomain every time the app starts. The address is therefore new on every
restart, and the tunnel itself is only promised for about a week. Nobody can
bookmark it.

This page can be bookmarked. It reads `status.json`, which the playground's
own watchdog rewrites and pushes whenever the address changes, and forwards
you to whatever is live now.

| File | Purpose |
| --- | --- |
| `index.html` | What people visit. Reads `status.json` and offers the link. |
| `status.json` | The current address and when it was last confirmed. |

`status.json` is written by `share_daemon.py` in the playground repository —
edit it there rather than by hand, or the next rotation will overwrite you.

## Why the password

The link on this page is public; the playground behind it is not. It runs on a
private API key, and anyone who reaches it spends that key's quota. Basic auth
is the only thing standing in the way, so the password is deliberately kept off
this page and passed to people directly.

## Delay after a rotation

Pages serves this directory from a CDN that holds a copy for ten minutes, so
the `status.json` sitting next to `index.html` is the stale one. The page
therefore reads it from `raw.githubusercontent.com` instead, which answers
from the repository and is purged when the watchdog pushes; the local copy is
only the fallback, for a network that blocks raw.

Some delay is still possible, which is why the page offers a link to click
rather than redirecting: a button that might be a minute out of date can say
so, where a silent redirect just drops you on a dead page.

`status.json` carries two times. `updated` is when the address went up, which
is what the page shows. `confirmed` is refreshed about once an hour while the
watchdog is running, and exists for the case it cannot report itself: a daemon
killed outright, or a machine that goes down, leaves whatever it last said
standing. When `confirmed` falls far enough behind, the page stops offering
the address rather than sending people to a link that can only fail.

Hourly rather than by the minute, which would mean a commit here every minute
forever.

