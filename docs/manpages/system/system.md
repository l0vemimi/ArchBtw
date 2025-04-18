# Init

## Systemd



### [Journal](https://wiki.archlinux.org/title/Systemd/Journal)

Journal configuration is in *etc/systemd/journald.conf* and writes to */var/log/journal*, **if this directory is deleted** systemd will instead write to /run/log/journal. The directory will be automatically recreated if *Storage=persistent* is added to *journald.conf*, *systemd-journald.service* is restarted, or the system is rebooted.

#### Clearing Journal Files

Journal files can be removed just using *rm*, additionally you can use *journalctl*.

Remove archived journals until the disk space falls below 100M:

        journalctl --vacuum-size=100M

Make all journal files contain no data older than 2 weeks:

        journalctl --vacuum-time=2weeks