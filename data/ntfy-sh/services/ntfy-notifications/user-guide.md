# ntfy Push Notifications — Tutorial

Push notifications delivered via HTTP. Publish from any script, CI pipeline, or app — receive on your phone, desktop, or via API.

---

## What You Get After Enrolling

After enrolling in **ntfy Push Notifications**, you receive:

- A unique **endpoint URL** like `https://api.unitysvc.com/ntfy/RZEVCL`
- The 6-character code (`RZEVCL`) is your **private topic** — all messages published to this URL are delivered only to subscribers of this topic

You'll also need your **UnitySVC API key** (found on the API Keys page in your dashboard) to publish messages through the gateway.

---

## Step 1 — Set Up the Mobile App

Install the official ntfy app so you can receive notifications on your phone.

| Platform | App |
|----------|-----|
| Android | [ntfy on Google Play](https://play.google.com/store/apps/details?id=io.heckel.ntfy) or [F-Droid](https://f-droid.org/packages/io.heckel.ntfy/) |
| iOS | [ntfy on App Store](https://apps.apple.com/us/app/ntfy/id1625396347) |

<img src="images/ntfy-app-store.jpg" alt="ntfy app store listing" width="300">

Once installed, add a subscription:

1. Tap **+** (or "Add subscription")
2. Set **Server URL** to `https://ntfy.svcpass.com`
3. Set **Topic** to your 6-character code (e.g. `RZEVCL`)
4. Tap **Subscribe**

<table><tr>
<td><img src="images/ntfyapp-settings.jpg" alt="App settings — set default server to ntfy.svcpass.com" width="300"></td>
<td><img src="images/ntfy-subscription.jpg" alt="Add subscription (iOS)" width="300"></td>
</tr><tr>
<td><em>Set default server to ntfy.svcpass.com</em></td>
<td><em>Enter your topic code and subscribe</em></td>
</tr></table>

<table><tr>
<td><img src="images/SubscribeToTopic.png" alt="Subscribe to topic (Android)" width="350"></td>
<td><img src="images/AfterSubscription.png" alt="Subscribed — ready to receive" width="350"></td>
</tr><tr>
<td><em>Subscribe to topic (Android)</em></td>
<td><em>Subscribed — ready to receive</em></td>
</tr></table>

> Mobile app subscriptions connect directly to the ntfy server (not through the UnitySVC gateway), so no API key is needed for receiving.

---

## Step 2 — Send Your First Message

Open a terminal and publish a plain-text message:

```bash
curl -X POST "https://api.unitysvc.com/ntfy/RZEVCL" \
  -H "Authorization: Bearer svcpass_YOUR_API_KEY" \
  -d "Hello from ntfy!"
```

You should see a JSON response confirming the message:

```json
{
  "id": "HwsCmea3Bz8H",
  "time": 1771019353,
  "event": "message",
  "topic": "RZEVCL",
  "message": "Hello from ntfy!"
}
```

Within seconds, the notification should appear on your phone.

<img src="images/serverURL_topicFields.png" alt="First notification received on desktop" width="600">

---

## Step 3 — Receive Messages

There are several ways to receive messages beyond the mobile app.

### HTTP JSON stream

Open a long-lived connection to stream messages in real time:

```bash
curl -s "https://api.unitysvc.com/ntfy/RZEVCL/json" \
  -H "Authorization: Bearer svcpass_YOUR_API_KEY"
```

This outputs newline-delimited JSON — one event per line — until you disconnect:

```json
{"event":"open","topic":"RZEVCL"}
{"event":"message","topic":"RZEVCL","message":"Hello!","title":"Test","priority":3}
{"event":"keepalive","topic":"RZEVCL"}
```

### WebSocket (Python)

```python
import asyncio
import json
import websockets

async def listen():
    uri = "wss://api.unitysvc.com/ntfy/RZEVCL/ws"
    headers = {"Authorization": "Bearer svcpass_YOUR_API_KEY"}

    async with websockets.connect(uri, additional_headers=headers) as ws:
        async for raw in ws:
            event = json.loads(raw)
            if event["event"] == "message":
                print(f"[{event.get('title', 'Notification')}] {event['message']}")

asyncio.run(listen())
```

### WebSocket (JavaScript)

```javascript
const ws = new WebSocket("wss://api.unitysvc.com/ntfy/RZEVCL/ws");

// Note: WebSocket API does not support custom headers in browsers.
// Use query parameter authentication if available, or use the
// HTTP JSON stream (EventSource) approach instead for browser apps.

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.event === "message") {
    console.log(`[${data.title || "Notification"}] ${data.message}`);
  }
};
```

### Event types

| Event | Description |
|-------|-------------|
| `open` | Sent when the connection is established |
| `message` | A published message (contains `message`, `title`, `priority`, `tags`, etc.) |
| `keepalive` | Periodic heartbeat (~30 seconds) to keep the connection alive |

---

## Step 4 — Priority, Tags, and Formatting

### Title, priority, and tags

ntfy supports rich notifications via HTTP headers:

```bash
curl -X POST "https://api.unitysvc.com/ntfy/RZEVCL" \
  -H "Authorization: Bearer svcpass_YOUR_API_KEY" \
  -H "Title: Build Failed" \
  -H "Priority: high" \
  -H "Tags: warning,build" \
  -d "Build #1234 failed on main branch"
```

### Priority levels

| Priority | Name | Use case |
|----------|------|----------|
| 1 | `min` | Background info, logs |
| 2 | `low` | Routine notifications |
| 3 | `default` | Normal messages |
| 4 | `high` | Important alerts |
| 5 | `urgent` | Critical — bypasses Do Not Disturb on mobile |

### JSON body format

You can also send structured JSON instead of using headers:

```bash
curl -X POST "https://api.unitysvc.com/ntfy/RZEVCL" \
  -H "Authorization: Bearer svcpass_YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Disk usage above 90%",
    "title": "Server Alert",
    "priority": 4,
    "tags": ["warning", "disk"]
  }'
```

### Click actions

Attach a URL that opens when the notification is tapped:

```bash
curl -X POST "https://api.unitysvc.com/ntfy/RZEVCL" \
  -H "Authorization: Bearer svcpass_YOUR_API_KEY" \
  -H "Click: https://example.com/dashboard" \
  -d "New report ready — click to view"
```

### Delayed delivery

Schedule a message for later:

```bash
curl -X POST "https://api.unitysvc.com/ntfy/RZEVCL" \
  -H "Authorization: Bearer svcpass_YOUR_API_KEY" \
  -H "Delay: 30m" \
  -d "Reminder: check on the deploy"
```

Accepts values like `30m`, `2h`, `tomorrow 9am`, or a Unix timestamp.

---

## Use Case Examples

### CI/CD pipeline notifications

Send a notification when a GitHub Actions workflow completes:

```yaml
# .github/workflows/deploy.yml
- name: Notify on success
  if: success()
  run: |
    curl -s -X POST "https://api.unitysvc.com/ntfy/RZEVCL" \
      -H "Authorization: Bearer ${{ secrets.UNITYSVC_API_KEY }}" \
      -H "Title: Deploy Succeeded" \
      -H "Tags: white_check_mark" \
      -d "Deployed ${{ github.sha }} to production"

- name: Notify on failure
  if: failure()
  run: |
    curl -s -X POST "https://api.unitysvc.com/ntfy/RZEVCL" \
      -H "Authorization: Bearer ${{ secrets.UNITYSVC_API_KEY }}" \
      -H "Title: Deploy Failed" \
      -H "Priority: high" \
      -H "Tags: x" \
      -d "Deploy failed for ${{ github.sha }}"
```

### Server monitoring

```bash
#!/bin/bash
USAGE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
if [ "$USAGE" -gt 90 ]; then
  curl -s -X POST "https://api.unitysvc.com/ntfy/RZEVCL" \
    -H "Authorization: Bearer svcpass_YOUR_API_KEY" \
    -H "Title: Disk Alert" \
    -H "Priority: high" \
    -H "Tags: warning" \
    -d "Disk usage at ${USAGE}% on $(hostname)"
fi
```

### Python application

```python
import requests

def notify(message, title=None, priority=None, tags=None):
    headers = {"Authorization": "Bearer svcpass_YOUR_API_KEY"}
    if title:
        headers["Title"] = title
    if priority:
        headers["Priority"] = str(priority)
    if tags:
        headers["Tags"] = ",".join(tags)

    requests.post(
        "https://api.unitysvc.com/ntfy/RZEVCL",
        headers=headers,
        data=message,
    )

# Usage
notify("Order #1234 shipped", title="Order Update", tags=["package"])
notify("Payment failed", title="Billing Alert", priority=4, tags=["warning"])
```

---

## Authentication

All requests through the UnitySVC gateway require your API key in the `Authorization` header:

```
Authorization: Bearer svcpass_YOUR_API_KEY
```

You can create API keys from the **API Keys** page in your UnitySVC dashboard.

---

## Endpoints Reference

All endpoints use your enrollment URL as the base. Replace `RZEVCL` with your topic code.

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/ntfy/RZEVCL` | Publish a message |
| `PUT` | `/ntfy/RZEVCL` | Publish a message (same as POST) |
| `GET` | `/ntfy/RZEVCL/json` | Subscribe via HTTP JSON stream |
| `GET` | `/ntfy/RZEVCL/ws` | Subscribe via WebSocket |

---

## Limits

- **Message size**: 4 KB max per message
- **Message retention**: 12 hours (messages older than 12h are discarded)
- **Topic isolation**: Each enrollment gets a unique topic code. You cannot publish to or subscribe to other enrollments' topics through the gateway.

---

## Troubleshooting

### 401 Unauthorized

- Verify your API key starts with `svcpass_` and is not expired or revoked
- Ensure the `Authorization: Bearer ...` header is included

### 404 Not Found

- Check that you are enrolled in the ntfy service (visit your Enrollments page)
- Verify the topic code in the URL matches exactly (codes are case-sensitive, uppercase only)

### WebSocket disconnects after ~5 minutes of inactivity

- The gateway keeps connections alive as long as data flows. ntfy sends keepalive messages every ~30 seconds, so this should not happen under normal conditions.
- If you see disconnects, check your client-side timeout settings.

### Messages not appearing on mobile

- Ensure you've entered the correct server URL (`https://ntfy.svcpass.com`) and topic code in the mobile app
- Messages are retained for 12 hours — older messages won't appear

---

## Further Reading

- [ntfy documentation](https://docs.ntfy.sh/) — Full ntfy protocol reference
- [ntfy publishing options](https://docs.ntfy.sh/publish/) — All supported headers and JSON fields
- [ntfy subscribe API](https://docs.ntfy.sh/subscribe/api/) — Subscription protocols and filters
