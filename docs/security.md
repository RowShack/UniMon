# Running UniMon safely

Back to the [README](../README.md).

UniMon holds the API keys of your UniFi consoles and, if you add it, your Site Manager (cloud) key, so treat its web page like an admin console.
This page says how to run it so that only the people you mean to can reach it.

## Best: keep it off the public internet

Reach UniMon over a VPN or a tailnet (WireGuard, Tailscale **without** Funnel, or your firewall's own VPN). Leave
`bind` on `127.0.0.1` (the default) or bind to the VPN address only, and keep the port closed to the internet. Clients
then arrive with their own addresses, so sign-in lockout and the audit log work as intended.

## If you publish it anyway

1. **Put an identity-aware gate in front** (a reverse proxy or access service that signs people in before a request
   reaches UniMon), ideally on a different machine.
2. **Tell UniMon which proxy to believe.** If the proxy runs on the same server (nginx, Caddy, a tunnel agent), add its
   address to `web.trusted_proxies` in `config.json` (or `trusted_proxies` on `POST /api/admin/web`) and make the proxy send
   `X-Forwarded-For` and `X-Forwarded-Proto`. Without this, every visitor looks like the proxy: a few wrong passwords from
   anyone can lock that one address out for 15 minutes. Forwarded headers from anyone not listed are ignored.
3. **Two-step sign-in for everyone.** Settings > Security > two-step sign-in required, and enrol every account.
   A server set up from another computer or listening on the network starts with it required for admins.
   Use long random passwords: an admin can download the UniFi API keys in a backup (the backup is protected by a
   password you choose, and asks for your password again).
4. **Finish the first-run setup yourself** before the port is reachable, and check Settings > Audit log for
   `login.failed` and `login.blocked`.
5. UniMon only ever sends read requests to a console and to the Site Manager. What else a UniFi API key could do
   depends on your console's version and the account that made it, so make the key as the [setup guide](setup.md) says,
   on the least-privileged account your console offers, and delete the key on the console if you think it leaked.

## What changed in 0.2.0

These were found by a security review of FortiMon, whose web server and sign-in UniMon shares, and were fixed for UniMon
in 0.2.0 (see the [changelog](https://github.com/RowShack/UniMon/releases)):

- Nothing beyond 4 KB is read from someone who isn't signed in, and requests with `Transfer-Encoding` or two
  `Content-Length` headers are refused.
- The first-run setup code is longer and limited to 5 wrong tries per address; setup without the code works only from the
  server itself.
- Sign-in lockout counts per address, per address and user name, and per user name, and the real owner can still get in
  from an address that signed them in recently.
- At most 256 connections (64 per address) and a 30-second request timeout; Linux service limit `LimitNOFILE=4096`.
- `Strict-Transport-Security` is sent over HTTPS.
- Backup with API keys, restoring users, making an admin, resetting an admin's password or two-step sign-in, and
  uploading a certificate ask for the password again (and an authenticator code if you use one).

## What was checked, and what was not

The security review itself was of FortiMon. For UniMon the shared parts were re-tested on a running UniMon demo
instance on localhost (every route against each role, setup through a proxy with and without `trusted_proxies`, lockout,
body and header limits, request framing, connection caps, two-step sign-in), the console and API-key routes were
checked to be admin only, and where a UniFi key can go was checked: a console key is sent only over HTTPS to a console
that shows exactly the certificate you trusted when you added it, a changed address needs the key typed again and the new
certificate trusted, no redirect is followed, and the Site Manager key goes only to `https://api.ui.com`, which is fixed
in the program and not a setting. Automated tests cover each fix.

One thing to know: an admin chooses the address of a console, so an admin can make UniMon open an HTTPS connection to any
address the server can reach and see whether it answers and which certificate it shows. Nothing else comes back, and no
key is sent before the certificate is trusted. Consoles live on private networks, so UniMon does not block private
addresses; keep admin accounts to people you trust with that.

It did **not** cover: behaviour behind a real reverse proxy or tunnel (only simulated headers), exhaustion at large scale,
a real UniFi console or the real Site Manager (fakes were used), the Windows service end to end, or any automated code-scanning tool. The Linux
service has no memory limit or syscall filter, and the `LimitNOFILE` setting reaches an existing Linux install only when you run
the `.run` installer again. Even with these fixes, exposing the port to everyone is not recommended.

Found a problem? Please report it privately: see [SECURITY.md](../SECURITY.md).
