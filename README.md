# bookingbrother-webapp

Telegram Mini App for the BookingBrother booking flow. Static HTML/JS — no build step, no server.

## What it is

A single `index.html` (~7 KB) that runs inside Telegram's WebView. The bot embeds the user's list of halls in the URL as a query param; the WebApp reads it, walks the user through hall → day → duration → time, and calls `Telegram.WebApp.sendData(JSON)` to send the selection back to the bot. The bot's Lambda receives it as a normal `Message.web_app_data` update.

## Deploy

Any static-hosting service with HTTPS works. Cheapest paths:

**Cloudflare Pages** (recommended — free, unlimited requests, one-click deploys)
1. Push this folder to a GitHub repo (e.g. `bookingbrother-webapp`).
2. Cloudflare dashboard → Workers & Pages → **Create application** → **Pages** → Connect to Git → pick the repo.
3. Build settings: framework preset = **None**, build command = *(blank)*, build output = `/`.
4. Deploy. Cloudflare gives you a URL like `https://bookingbrother-webapp.pages.dev`.

**GitHub Pages** (also free, slightly slower to propagate)
1. Push this folder to a repo.
2. Repo → Settings → Pages → Source = `main` branch, `/` root.
3. Wait for the URL: `https://<user>.github.io/<repo>/`.

**S3 + CloudFront** (few cents/month, keeps everything in your AWS account)
1. `aws s3 mb s3://bookingbrother-webapp`
2. `aws s3 sync . s3://bookingbrother-webapp --exclude "README.md"`
3. Attach a CloudFront distribution with the bucket as origin. Enable HTTPS.

Whatever URL you end up with, set it on the bot Lambda:

```
WEBAPP_URL = https://bookingbrother-webapp.pages.dev
```

## Wire the URL to Telegram (optional but nice)

Once the URL is live, register it as the bot's Mini App in @BotFather:

```
/newapp
  ↓
  pick your bot
  ↓
  title: BookingBrother
  short name: book
  photo: (any 640×360 square)
  URL: https://bookingbrother-webapp.pages.dev
```

BotFather returns a link like `https://t.me/YourBot/book` you can share. The `/book` command in the bot already opens the WebApp inline via a reply-keyboard button, so this is optional.

## Files

- `index.html` — the whole app. Vanilla JS, no dependencies except `telegram-web-app.js` loaded from `telegram.org`.

## Debug

To open the WebApp outside Telegram (e.g. to iterate on the UI in a normal browser), append `?halls=` with a fake JSON payload:

```
https://your-webapp/?halls=%5B%7B%22id%22%3A%22H1%22%2C%22city%22%3A%22Bratislava%22%2C%22address%22%3A%22Hall%20A%22%7D%5D
```

Which is the URL-encoded form of `?halls=[{"id":"H1","city":"Bratislava","address":"Hall A"}]`. The Telegram-specific bits (theme vars, MainButton, `sendData`) won't work but you can see the layout.

Inside Telegram, use [@BotFather](https://t.me/botfather) → `/mybots` → *your bot* → **Bot Settings** → **Configure Mini App** to see logs, or enable "WebApp Inspector" from Telegram Desktop → Settings → Advanced → Experimental settings.
