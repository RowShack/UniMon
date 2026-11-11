# UniMon FAQ

Back to the [README](../README.md).

## Does UniMon change anything on my console?

No. It uses a read-only API key and only reads.

## Where is my API key kept?

Encrypted in `config.json`, with a key stored beside it (`unimon.key`; protected by Windows DPAPI on Windows, readable only by
the service user on Linux). It is never shown again.
## Why does UniMon ask me to trust a certificate?

UniFi consoles use self-signed certificates. UniMon shows the console's fingerprint without sending the key, so you can
compare it with the console's own. Once trusted, UniMon talks only to that exact certificate.

## A console shows "not connected"

Open **Settings > Consoles** and press **Test** on it. The status line gives the reason. Check the address and port, that the
key is a Network Integration API key (not a password), and that the certificate has not changed.

## Why is my console's WAN health labelled unofficial?

A gateway's own API has no official WAN health call. UniMon uses the legacy `stat/health` data and labels it as unofficial.
Add a Site Manager key for Ubiquiti's official ISP data (latency, loss and ISP).

## Why is the Changes tab built from comparisons?

UniFi has no change log to read. UniMon keeps each reading and compares it with the next, so it can only show changes that
happen while it is monitoring.

## Does it work with several sites on one console?

It is built to, and the demo shows it, but it is **not verified on real gear with several sites yet**.

## I forgot the admin password

On the server itself, run `--reset-password admin` (see [configuration](configuration.md#users-and-roles)).

## How do I update?

**Settings > Updates** checks once a day, and a service install can use **Update now**. Otherwise run the newer installer
over the old one: data and settings are kept.

## Can it feed SiteMon?

Yes. Make a **card** key under **Settings > API keys** and add UniMon in [SiteMon](https://github.com/RowShack/SiteMon), or use
**Send to SiteMon** on the same page if SiteMon cannot reach this server.
