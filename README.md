# Argo Alert — remote dashboard

One HTML file. It subscribes to the panel's MQTT topics over a secure
WebSocket and draws the same room cards you see on the panel itself. Open it
locally, or host it anywhere static: GitHub Pages, Cloudflare Pages, Netlify.

There is no server and no build step.

## Connecting it

Open the page, press **Broker**, and fill in:

| Field | Value |
|---|---|
| WebSocket host | your HiveMQ cluster address |
| Port | 8884 |
| Path | /mqtt |
| Username, Password | a HiveMQ access credential |
| Tenant | must match `CLOUD_TENANT` in the firmware |
| Device filter | a device name, or `+` to watch every panel of that tenant |

Port 8883 is for the panel. Browsers cannot speak raw MQTT, so the page uses
8884 with the WebSocket path. Both reach the same broker.

Settings are remembered in the browser, so a return visit connects by itself.

## What it shows

Retained topics mean the page is correct the instant it connects, without
waiting for the panel to send anything. Room names and timings come from the
config topic, current state from the state topic, and whether the panel is
alive from its status topic, which the broker maintains through the panel's
last will.

If a panel drops, the page says so and disables the buttons, because a command
sent to an absent panel would silently do nothing.

Accept and Clear publish to the panel's command topic. The panel echoes the
result as an event marked remote, so the activity list distinguishes a nurse at
the panel from someone acting from a browser.

## The gap you must close before customers use this

Every viewer currently types the same broker credential, and it lives in their
browser. That is fine while the only viewer is you. It is not fine for a
product: anyone with that credential can subscribe to, and publish to, every
topic it is permitted.

Stage two replaces it with a login against your own backend, which issues a
short-lived per-user credential scoped to that customer's topic prefix. The
broker then enforces isolation regardless of what the page does.
