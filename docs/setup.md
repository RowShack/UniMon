# UniMon setup guide

This takes you from nothing to a dashboard showing your first UniFi console. Back to the [README](../README.md).

UniMon reads each console with a **read-only API key**, so it can look but not change anything. It supports API keys only;
there is no local-account sign-in.

## Requirements

- A **Windows** server (64-bit) or a **Linux** server with systemd and Python 3.9 or newer (Ubuntu, Debian, RHEL, Rocky
  and similar). The Windows installer includes everything it needs.
- A web browser. UniMon serves its dashboard over HTTPS, on port 7443 by default.

## Install

Download the newest files from the [latest release](https://github.com/RowShack/UniMon/releases/latest).

### Windows

1. Download `UniMon-<version>.msi` and `SHA256SUMS.txt`.
2. [Check the download](#check-the-download).
3. Double-click the MSI and answer the questions:

   | Question | Choices |
   |---|---|
   | **Run UniMon** | **As a Windows service** (recommended): starts with Windows, keeps monitoring when nobody is signed in and restarts itself if it stops. Or **as a program** that runs only while it is open. |
   | **Who can open the web interface** | **Other computers on the network** (adds a Windows Firewall rule) or **only this computer**. |
   | **Port** | 7443 by default (HTTPS). |

   At the end it opens UniMon in your browser, where you create the first admin account.

Silent install (Intune, GPO, an RMM tool):

```
msiexec /i UniMon-<version>.msi /qn MODE=service ACCESS=all PORT=7443
```

`MODE` is `service` or `app`, `ACCESS` is `all` or `local`. An upgrade keeps the earlier choices and all data: install the
newer MSI over the old one.

The service is called **UniMon** in `services.msc`. Its data (settings, users, history, certificates, logs) is in
`C:\ProgramData\UniMon`. Uninstalling keeps that folder; delete it yourself to remove everything.

**Windows may warn you.** The installer is not signed with a paid code-signing certificate yet, so SmartScreen can show
"Windows protected your PC". Check the SHA-256 below first. If it matches, choose **More info**, then **Run anyway**.

### Linux

```
sudo sh unimon-<version>-linux.run
```

It asks whether to install a systemd service (recommended), the port, whether other computers may connect and, if ufw or
firewalld is active, whether to open the port. Run a newer `.run` over the top to upgrade; data is kept.

| | |
|---|---|
| Program | `/opt/unimon` |
| Data | `/var/lib/unimon` |
| Service | `systemctl status unimon`; logs with `journalctl -u unimon -f` |
| Command | `sudo unimon --help` runs UniMon's commands against the service's data |
| Uninstall | `sudo /opt/unimon/uninstall.sh` (add `--purge` to delete the data too) |

For no questions: `sudo sh unimon-<version>-linux.run --unattended --service --port 7443`. `--local-only`,
`--no-firewall` and `--no-service` also work; `--help` lists everything.

### Check the download

Each release lists `SHA256SUMS.txt`. The checksum of your file must match its line there.

```
# Windows (PowerShell)
Get-FileHash .\UniMon-<version>.msi -Algorithm SHA256

# Linux
sha256sum -c SHA256SUMS.txt --ignore-missing
```

The in-app updater goes further: it only accepts releases signed with MonStack's own key, which is built into the app.

## First sign-in

Open `https://<server>:7443/`. The first time, UniMon asks you to **create the first admin account**. If you create it
from another computer rather than on the server itself, it asks for a setup code, which is in `setup-code.txt` in the data
folder and in the log. UniMon makes its own self-signed HTTPS certificate, so browsers warn the first time.

## Make an API key on the console

A console is a UniFi gateway or Cloud Key that runs the Network application. In the UniFi Network application open
**Settings > Control Plane > Integrations** and create an **API key**. It is read-only.

## Add a local console

1. In UniMon open **Settings > Consoles > + Add a local console**.
2. Enter the console's address and the key, then **Test**.
3. UniMon first shows the console's certificate fingerprint **without sending the key**. Compare it with the console's own
   and trust it. From then on UniMon only talks to that exact certificate.
4. Each console's sites appear as sites in UniMon within a minute (it reads every 60 seconds by default).

## Add a console through the Site Manager (optional)

For a console that is not on this network, or to get the official WAN health (ISP, latency and loss):

1. Make an API key at `unifi.ui.com` under API (the Site Manager key).
2. In **Settings > Consoles** save it under **UniFi Site Manager**.
3. Use **+ Add through the Site Manager** and pick the console from the list.

Without a Site Manager key UniMon uses each console's own API only.

## Try it without UniFi gear

`unimon --demo` (or `python unimon.py --demo` from source) runs three simulated consoles. It keeps its own settings and
users.

## Next steps

Create users, set up alerts and reports, and put UniMon's address under **Settings > Web server & HTTPS** so links in alerts
open the right page: see the [configuration reference](configuration.md).
