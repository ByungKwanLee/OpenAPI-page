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

The same `status.json` can be read three ways, and none is both fresh and
free:

| | how current | how often it can be asked |
| --- | --- | --- |
| `api.github.com` | immediately | 60 requests an hour per address |
| `raw.githubusercontent.com` | up to 5 minutes behind | freely |
| this directory, via Pages | up to 10 minutes behind | freely |

Cache-busting does not rescue the middle one: the identical request comes back
with a different age depending on which edge server answers. So the page uses
the API where it has budget — on load, which is when someone has just started
the playground and wants to see it — and raw for the polling in between.

That is also why the page offers a link to click rather than redirecting: a
button that might be a couple of minutes out of date can say so, where a
silent redirect just drops you on a dead page.

## How it knows the playground has stopped

A watchdog shut down properly says so, and `status.json` goes to `offline`
within seconds. One that is killed outright — or a machine that loses power —
says nothing, and the last address it published would sit here looking live.

So the page checks. Before offering the link it fetches it, which works
because the two cases differ where a browser can see them: a live tunnel
answers with an `access-control-allow-origin` header, and a subdomain that has
been reclaimed answers 404 without one, so the fetch is refused. That makes
the page correct the moment the tunnel goes, with no heartbeat to maintain and
no commits to spend on one.

