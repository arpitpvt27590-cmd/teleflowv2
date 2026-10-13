# TeleFlow — Telegram forwarding dashboard

A self-hosted browser dashboard for forwarding Telegram messages using Telethon (a Telegram user session). Includes multiple Telegram account sessions (each with its own API ID, API hash, and phone number), source/destination chat selection, live forwarding, optional recent history (up to 1,000 messages per rule), a per-rule 0–120 second delay slider, per-rule text/link prefixes and suffixes, bot/word/domain filters, and text replacements.

## Important before deploying
- This is an application source bundle, **not an already-hosted website or a prebuilt Docker image**. Render's Docker Image URL field cannot use this ZIP directly; a Docker image must first be built and published to a registry.
- Render free web services can sleep and their filesystem is ephemeral. For persistent Telegram sessions/config, attach a persistent disk at `/data` (if your Render plan permits it) and use a service that stays running. Otherwise this is not dependable for 24/7 forwarding.
- Keep this private. A Telegram user session can access the chats available to that account. Never publish `/data`, session files, API hash, or login codes.
- Follow Telegram's terms and only forward content you have permission to redistribute. Forwarding can be subject to rate limits and privacy restrictions.

## Environment variables
Set these in the hosting service:
- `ADMIN_PASSWORD`: strong dashboard password.
- `TG_API_ID` and `TG_API_HASH` are optional legacy defaults. New account connections ask for an API ID and API hash separately for each Telegram account. Get these from https://my.telegram.org.
- `SESSION_SECRET`: long random secret used to sign the dashboard session cookie.
- `DATA_DIR=/data`
- `PORT=10000` (Render commonly supplies `PORT` automatically).

## Build and run locally with Docker

```sh
docker build -t teleflow-dashboard .
docker run --rm -p 10000:10000 \
  -e ADMIN_PASSWORD='change-this-password' \
  -e TG_API_ID='your-id' \
  -e TG_API_HASH='your-hash' \
  -e SESSION_SECRET='use-a-long-random-value' \
  -v teleflow-data:/data teleflow-dashboard
```
Then open http://localhost:10000.

## How to deploy to Render using a Docker image URL
1. Put these source files in a private GitHub repository.
2. Build the Docker image and publish it to a container registry (for example GitHub Container Registry, GHCR). The image must be accessible to Render; if private, configure registry credentials in Render.
3. In Render, create a **separate** web service using the published image URL. Do not replace the existing `rdpgotty` image until the new service has been tested.
4. Add the environment variables above. Set a persistent disk mounted at `/data` if available on your plan.
5. Open the new service URL and sign in with `ADMIN_PASSWORD`.
6. For each Telegram account, create/get an API ID and API hash at https://my.telegram.org. In the dashboard, enter that account’s API ID, API hash, and phone number, then enter the login code. The code may arrive inside Telegram rather than SMS. The code may arrive inside Telegram rather than SMS. If two-step verification is enabled, enter the Telegram password in the dashboard.
7. Create a route by choosing the source and destination chat IDs. Each route has its own delay slider (0–120 seconds) and a Message format section to add text or links before/after each forwarded post. Set the delay to 0 for immediate forwarding. Changes are saved per route.

## Known limitations
- Historical forwarding processes up to 1,000 messages per rule in the background. Test on a small source chat first.
- Filters are global across routes; blocked domains currently match text anywhere in the message/caption.
- This is a starter app; use a persistent disk and monitor logs. Do not assume a free Render service is a 24/7 worker.
