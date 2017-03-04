# APTREMOTEMUX

aptremotemux is inspired by [hostmux](https://github.com/hukl/hostmux).

It reads the [check_mk Multisite](https://mathias-kettner.de/check_mk.html) status of the APT check and creates a tmux session with windows and panes. Then it starts a SSH session for each found due APT update and executes an update so you can monitor the update and interact if necessary. If it detects that a host is on a remote site first a SSH session to the status host is initiated and form the status host to the target host where the update is started. The target host can use the key of the status host or your personal key by agent forwading (Option: -A). Update could be fully automated (Option: -y) and the SSH session can be closed if the command was sucessfull (Option: -e).

## REQUIREMENTS

aptremotemux relies on:

* tmux 2.3 + with earlier versions naming of panes will not work
* [libtmux](https://github.com/tony/libtmux)
* Python modules: urllib2, json, os, argparse, sys
* A custom WATO view
* A automation user called "auto" which has access to the custom WATO view

See: [WATO Web-API](https://mathias-kettner.de/checkmk_wato_webapi.html) and [Multisite Automation and Web Service](https://mathias-kettner.de/checkmk_multisite_automation.html)

Most important for automation: SSH keys have to be installed on all hosts to allow login wihtout password.

## WATO VIEW

aptremotemux relies on a custom WATO view called "aptupdates" with the following properties:

```
Make this view available for all users
Datasource: All services
Column: Site ID
Column: Hostname
Context / Search Filters: Column: APT Updates
Service states: WARN, CRIT
Problem acknowledged: no
Service in notification period: ignore
Host/service in downtime: no
Host: UP, PENDING
Is summary host: no
```

## CONFIGURATION

aptremotemux uses a config file. The default name and location of the config file is:

```
/usr/local/etc/aptremotemux.conf
```

Is contains pain Python varialbe definitions and it looks like this:

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

### CONFIG

MKPROT = "https"                  # HTTP/HTTPS - Prorocol uesed by Multisite site
MKHOST = "nagios.server.de"       # Multisite hostname
MKSITE = "MHC"                    # Multisite site name
MKUSER = "auto"                   # WATO automation user see:
                                  # https://mathias-kettner.de/checkmk_multisite_automation.html
MKPASS = "11122233344455566688"   # WATO automation user secret
TERM   = "xterm"                  # TERM to be used in SSH sessions

# check_mk site IDs and targets for SSH as there is no way to get a list from WATO

SITES = {
"SITEID":"ssh.target.host",
}

# local domains, hostnames get striped as used in WATO

LOCDOMS = [
".example.com",
]

# hosts to ignore as defined in WATO

IGNORE = [
"not.me.example.com",
]
```

## USAGE

```
usage: aptremotemux [-h] [-l] [-y] [-A] [-e] [-d] [-p PANES] [-c CONFIG]
                    [-C COMMAND]

Automats SSH connections in TMUX windows and panes for APT updates based on
the status of the APT check of check_mk.

optional arguments:
  -h, --help            show this help message and exit
  -l, --list            only list commands, do nothing
  -y, --yes             use apt with -y for automation
  -A, --agent           use ssh with -A for agent forwarding
  -e, --exit            auto exit after upgrade if exit code is zero
  -d, --debug           enabel console debug logging
  -p PANES, --panes PANES
                        maximum number of panes per window, default: 6
  -c CONFIG, --config CONFIG
                        path to config filei, default:
                        /usr/local/etc/aptremotemux.conf
  -C COMMAND, --command COMMAND
                        APT command to use, default: dist-upgrade
```
