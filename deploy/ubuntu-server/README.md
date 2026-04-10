# Ubuntu Server Deploy

This folder prepares a minimal OpenClaw deployment for an Ubuntu server.
It assumes:

- the gateway runs on the Ubuntu host
- you connect from your own machine over SSH tunnel
- model secrets live on the server, outside git

## Files

- `.env.server.example`: environment template for `~/.openclaw/.env`
- `openclaw.server.json`: minimal gateway config for `~/.openclaw/openclaw.json`

## Where to put tokens

Put provider and gateway tokens on the Ubuntu server in:

```bash
~/.openclaw/.env
```

For your current plan, fill at least:

```env
OPENCLAW_GATEWAY_TOKEN=your-long-random-token
DEEPSEEK_API_KEY=your-deepseek-api-key
```

If you switch providers later, use the matching env var instead:

- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`
- `GEMINI_API_KEY`
- `OPENROUTER_API_KEY`

## Recommended server setup

1. Install Node 24 and pnpm on the Ubuntu server.
2. Copy your OpenClaw fork or working tree to the server.
3. Build the project:

```bash
pnpm install
pnpm ui:build
pnpm build
```

4. Create the runtime directory and copy the templates:

```bash
mkdir -p ~/.openclaw
cp deploy/ubuntu-server/.env.server.example ~/.openclaw/.env
cp deploy/ubuntu-server/openclaw.server.json ~/.openclaw/openclaw.json
```

5. Edit `~/.openclaw/.env` and replace placeholder values with real tokens.

6. Install the gateway service:

```bash
pnpm openclaw onboard --install-daemon
```

If you prefer the direct command path:

```bash
pnpm openclaw gateway install
```

## Manual test run before daemon install

Run this once on the server to confirm the config loads:

```bash
pnpm openclaw gateway --port 18789 --verbose
```

The gateway should stay bound to loopback. Access it from your laptop through an SSH tunnel:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@your-server
```

Then open:

```text
http://127.0.0.1:18789/
```

Authenticate with `OPENCLAW_GATEWAY_TOKEN`.

## Notes

- Keep real secrets out of git.
- Keep `gateway.bind: "loopback"` unless you deliberately deploy behind a trusted reverse proxy or tailnet setup.
- Add channels only after the base gateway is healthy.
- If you later need Telegram or Discord, add the token to `~/.openclaw/.env` and then add the matching channel block in `~/.openclaw/openclaw.json`.
