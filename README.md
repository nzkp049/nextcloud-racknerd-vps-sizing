# Nextcloud VPS hosting: Which RackNerd KVM plan actually runs it, what the yearly price is, and what you still install yourself (plan table + backup checklist)

RackNerd does not sell a pre-installed Nextcloud VPS. Its one-click catalog in the ctrl control panel covers AI and agent tools, n8n, Coolify and a couple of server panels; Nextcloud is not on that list, so you install it over SSH like any other LAMP application. On the yearly KVM specials that puts the practical starting point at the **4 GB plan for $59.99/year** (60 GB SSD, 2 vCore, 7 TB at 1 Gbps). RackNerd's own setup guide names 2 GB of RAM as the floor, which leaves almost nothing once MariaDB, Redis and image previews are running.

You can read this in two ways: RackNerd costs less per year than most managed Nextcloud templates, and you do the setup work yourself. The rest of this page covers the plans, the software minimums that decide whether a small box survives, where Office and Talk break, and the backup job that outlives the install.

> **Most readers:** the 4 GB KVM VPS at $59.99/year handles files, calendars and contacts for a household or a team of 1–5.
> **10+ active users, or browser-based Office editing:** the 6 GB KVM VPS at $89.99/year, and budget an evening for Collabora.
> 👉 [Check the current RackNerd KVM VPS specials and pick a datacenter](https://bit.ly/RacKnerd)

## RackNerd KVM VPS plans for Nextcloud

| Plan | RAM / vCPU | Storage | Bandwidth | Price and term | Order |
| --- | --- | --- | --- | --- | --- |
| 4 GB KVM VPS | 4 GB / 2 vCore | 60 GB SSD | 7,000 GB @ 1 Gbps | $59.99/year | [4 GB KVM VPS](https://bit.ly/RacKnerd) |
| 6 GB KVM VPS | 6 GB / 6 vCore | 100 GB SSD | 12,000 GB @ 1 Gbps | $89.99/year | [6 GB KVM VPS](https://bit.ly/RacKnerd) |

Both include a dedicated IP, full root SSH and 24x7 support. RackNerd rotates its specials, so the order page is the live list, and that is also where you confirm the term before paying. Storage is the number that runs out first on a file cloud: 60 GB fills quickly if you sync phone photos, and Nextcloud's External Storage app lets you mount S3 or SFTP later rather than upgrading the whole box.

Every plan includes the ctrl panel: OS reinstall, an HTML5 VNC console, root password reset, a network firewall and one-click apps. RackNerd lets you pick CentOS, Rocky Linux, AlmaLinux, Fedora, Debian or Ubuntu at checkout, and custom ISO installs go through a support ticket. The image you want is Ubuntu or Debian, because Nextcloud's documentation and most community guides assume it.

Docker matters more than it looks here. ctrl installs Docker with Portainer in one click, which is the difference between "Collabora is possible" and "Collabora is impossible" on hosts that block containers.

## What Nextcloud actually requires before you buy RAM

Nextcloud's current manual asks for a 64-bit Linux server, PHP 8.3 to 8.5, MariaDB 11.4 and Apache 2.4. Those are version floors, and a distro that ships PHP 8.1 will fail before you upload a single file. Ubuntu with a recent PHP branch and MariaDB covers it; check `php -v` after install rather than assuming.

Memory is specified per PHP process, not per server: at least 128 MB each, 512 MB recommended. A handful of concurrent sync clients, the web UI and a cron job add up to several workers, and the built-in updater alone wants 256 MB of PHP memory to run. That arithmetic is why a 1 GB or 2 GB box feels fine on day one and stalls on day thirty. The 4 GB tier exists to absorb that overhead, not because Nextcloud is heavy by itself.

One licensing detail that gets repeated badly: the 100-user limit belongs to the Docker All-in-One image only. A manual install of the AGPL software has no user cap and no per-user fee. If you are avoiding the AIO route anyway because you want a plain LAMP stack, user count is a resource question, not a licensing one.

## Sizing by users, not by benchmark screenshots

Third-party sizing tests put a 4 GB box at 1–5 users with files, calendars and contacts; 8 GB around 10–25 users once Redis and image previews are enabled; and 16 GB near 100 users with full-text search running. Treat those as rough guides rather than vendor promises.

Against RackNerd's two plans that translates to:

- **1–5 people, mostly documents and photos under 50 GB:** 4 GB KVM VPS. Runs Nextcloud, MariaDB and Redis with room to spare.
- **6–15 people, desktop and mobile clients syncing constantly, previews turned on:** the 6 GB plan. The extra RAM matters less than the 100 GB disk, since previews and versions eat space fast.
- **More than 15 heavy users, or you plan to run Collabora on the same box:** price a larger special from the current list rather than squeezing two services into 6 GB.

👉 [Order the 4 GB KVM VPS at $59.99/year for a small household setup](https://bit.ly/RacKnerd)

## Nextcloud Office and Talk need a second service, and that changes the math

Installing Nextcloud gives you files, sharing, calendars and contacts. It does not give you Google-Docs-style editing or group video calls. Nextcloud Office needs a Collabora Online or OnlyOffice server sitting next to your instance, and Talk's high-performance backend is a separate service with its own memory footprint.

This is where a cheap VPS plan stops being cheap in time rather than money. Collabora ships as a Docker container, which is why RackNerd's one-click Docker and Portainer install is worth knowing about: you can run CODE (the Collabora development edition) on the same VPS, put it behind an nginx virtual host with a Let's Encrypt certificate, then point the Nextcloud Office app at it. It works. It also competes with Nextcloud for RAM.

OVHcloud's guidance for its own Nextcloud VPS is revealing: 8 GB is the minimum recommended, and 12 GB for teams using Office or Talk simultaneously. That is a vendor selling the same software stack arriving at the same conclusion you will reach after trying to run Collabora on 4 GB.

If document editing is the main reason you want Nextcloud, start at the 6 GB plan and keep an eye on memory during the first week. If it is file sync and calendars with an occasional shared spreadsheet, skip Collabora entirely and save the resources.

👉 [Take the 6 GB KVM VPS if Collabora or Talk is on your roadmap](https://bit.ly/RacKnerd)

## Backup and upgrade work that nobody puts in the sales page

Persistent storage is not a backup. LightNode states this outright about its own Nextcloud template, and it applies to any VPS: disks survive reboots, they do not survive a bad `occ` command or a deleted data directory. RackNerd is unmanaged, so this is entirely yours.

A complete Nextcloud backup covers four things:

1. The `config` directory.
2. The `data` directory where user files live.
3. The `custom_apps` directory, if you installed apps outside the default set.
4. A database dump of the Nextcloud schema.

External mounts are not in that list. If you attached an S3 bucket or an SFTP folder through the External Storage app, back up the bucket separately, because restoring Nextcloud alone will leave empty folders pointing at nothing. The same trap applies to primary object storage: switching it on later makes existing local files unreachable, so decide before you upload data.

For upgrades, Nextcloud offers a web updater at `/settings/admin/overview` and a CLI path using `occ upgrade` after replacing the application files. Take a snapshot first, and move up one major version at a time. Skipping versions is the most common way to turn a routine update into a restore.

## Version currency is part of the security posture

Nextcloud shipped three Hub releases this year, and Hub 25 Autumn no longer receives security fixes. Running an unsupported release on a box with a public IP and your documents on it is the risk to plan around, and it is why the update job deserves a calendar reminder rather than good intentions.

Two cheap habits on top of that: run Nextcloud's security scan at `scan.nextcloud.com` after you finish the HTTPS setup, and turn on two-factor authentication for the admin account. RackNerd's network firewall in ctrl can restrict SSH to your own address, which removes a large share of the noise in your auth log.

## How RackNerd compares on price

- **OVHcloud:** VPS-2 from $8.50/month with 4 vCores, 8 GB RAM and 75 GB NVMe, and daily backups included with 24-hour retention. Monthly billing, backups handled for you, roughly $102 in the first year.
- **Hostinger:** KVM 1 at $6.49/month, but only as a 24-month prepay of $155.76; it renews at $11.99/month, which is about $144 in year two. Ships a pre-installed Nextcloud template and free weekly backups, and states plainly that no technical support is included.
- **LightNode:** Nextcloud template from $13.7/month for 4 GB, 2 vCPU and 50 GB NVMe, with the caveat that persistent storage is not automatic backup.
- **Contabo:** free pre-installed Nextcloud with unlimited users; its advertised prices are first-24-month promotional rates, so the renewal figure is the one that matters.
- **GreenGeeks:** the plan cards show $9.99/month for 4 GB while the FAQ on the same page quotes $69.95/month for the same tier. When a page disagrees with itself, do not build a budget on it.

RackNerd's advantage is the yearly price and full root on a 1 Gbps port with a choice of locations. Its disadvantage is that nothing is managed: no Nextcloud template, no update service, no backup.

## Which plan to buy

Files, calendars and contacts for a handful of people: **4 GB KVM VPS at $59.99/year**. The 2 vCore and 60 GB are enough until previews and file versions pile up, and the External Storage app lets you push archives to S3 instead of upgrading.

A team that will actually open documents in the browser, or run Talk with its high-performance backend: **6 GB KVM VPS at $89.99/year**, and expect to spend real time on Collabora configuration. If more than 15 people sync daily, price a bigger special from the current list instead of tuning a small box.

Before checkout, confirm four things: the billing term (these are yearly prepays, not monthly), what the renewal rate will be, which datacenter is closest to the people syncing files, and whether you need IPv6, which RackNerd provides on request in Los Angeles and France through a support ticket. Then decide how you will back up the config, data, custom apps and database, because that part is not included with any unmanaged VPS at this price.

👉 [Start with RackNerd's current KVM VPS specials](https://bit.ly/RacKnerd)
