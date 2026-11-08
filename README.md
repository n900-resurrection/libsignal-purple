# libsignal-purple

A libpurple protocol plugin that shows [Signal](https://signal.org) as an ordinary account in the stock messaging UI of the
Nokia N900 (Maemo 5): Settings > Accounts and the Conversations application, through telepathy-haze. The plugin itself does
not speak Signal; it talks over a unix socket to the `signal-bridge` daemon (separate repository, Rust, built on presage),
which uses the account of a device already linked with gurk.

```
Conversations / Pidgin -> telepathy-haze -> libsignal-purple.so (this repo) -> unix socket -> signal-bridge -> Signal servers
```

## What works (2026-11-08, first test on a phone)
- the account (no password, any name), contacts as buddies with aliases, your own contact as "(Note to Self)";
- sending and receiving text messages, messages written on another device (shown as sent), the history received while offline;
- if the daemon is not running the plugin starts it (`/opt/signal-bridge/bin/signal-bridge-start`) and waits; without network the
  login ends with an error after 120 s.

Not yet: groups, attachments, typing and read notifications, new contacts by number. The safety guard `--allow` of the bridge
decides who may receive messages from the phone (the project owner's own account is used for tests).

## Package
`make debs APP=libsignal-purple` in the N900 project builds the .deb. It installs the plugin into `/opt/libsignal-purple/lib`
(symlink in `/usr/lib/purple-2`), adds a `[Protocol signal]` block (markers `BEGIN/END libsignal-purple`, a backup is made) to
`/usr/share/telepathy/managers/haze.manager`, an account profile `/usr/share/osso-rtcom/signal.profile` and an icon. Mission
Control reads the manager files at start, so reboot after installing; `dpkg -r libsignal-purple` reverts the changes.
Depends on `libpurple0`, `telepathy-haze` and `signal-bridge`. Tests on the phone: `make -C apps/libsignal-purple test`.

Design notes and the protocol of the socket: `DOC/signal-native.md` in the N900 project.

License: GPL-2.0-or-later (the plugin is loaded into libpurple and uses its headers).
