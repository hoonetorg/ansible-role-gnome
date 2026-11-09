# ansible-role-gnome

GNOME settings for all users on openSUSE Leap, from the inventory: dconf system defaults and locks, default
applications. A new setting is inventory data, no role code.

1. installs `dconf`
2. writes the dconf profile `/etc/dconf/profile/user` (`user-db:user`, `system-db:local`): every user reads
   their own settings first, then the system db `local`
3. writes `/etc/dconf/db/local.d/50-ansible` from `settings` (checked with `dconf compile` before it replaces
   the file: an invalid value stops the run and names key and value)
4. writes `/etc/dconf/db/local.d/locks/50-ansible` from `locks`
5. on changes: `dconf update` (compiles `/etc/dconf/db/local`)
6. writes `/etc/xdg/gnome-mimeapps.list` from `default_apps` (only this role writes it)

Without `settings` / `locks` / `default_apps` the role removes its own files again (desired state); the profile
stays (harmless). Users see changed settings after logging in again, changed default applications at once.

**Default or lock:** a setting alone is only the default - a user who already stored their own value (GNOME
writes some at first login, e.g. the keyboard layouts) keeps it. A locked key always has the system value, also
for those users, and users can't change it.

**Default applications:** openSUSE generates `/usr/share/applications/gnome-mimeapps.list` from
`/etc/gnome_defaults.conf`; for MIME types that file names no application for, it picks one of the installed
ones (Leap 16.1: PDF → LibreOffice Draw). `/etc/xdg/gnome-mimeapps.list` beats it in GNOME and GNOME Classic
sessions (XDG mimeapps spec, desktop-specific file). Only a default: a user's own choice ("Open With…" → set as
default, stored in `~/.config/mimeapps.list`) wins. Other desktops are not affected.

Only openSUSE is implemented. Per-user settings (`community.general.dconf`) are not implemented yet.

## Variables

```yaml
gnome:
  settings:                          # dconf path (no leading/trailing slash) → key → value (GVariant)
    org/gnome/desktop/input-sources:
      sources: "[('xkb', 'us'), ('xkb', 'de+nodeadkeys')]"   # us = default, Super+Space switches
    org/gnome/desktop/interface:
      clock-show-seconds: true       # booleans and numbers as is
      # strings need inner quotes:   color-scheme: "'prefer-dark'"
  locks:                             # full key paths users can't change
    - /org/gnome/desktop/input-sources/sources
  default_apps:                      # MIME type → desktop file id (/usr/share/applications/<id>)
    application/pdf: org.gnome.Evince.desktop
```

Find MIME types and applications: `gio info -a standard::content-type <file>`, `gio mime <type>` (in the GNOME
session: shows the default and all candidates).

Find keys and values: `gsettings list-recursively org.gnome.desktop.input-sources`, or change the setting in
GNOME Settings and watch `dconf watch /`.

## Check

```bash
cat /etc/dconf/db/local.d/50-ansible /etc/dconf/db/local.d/locks/50-ansible
gsettings get org.gnome.desktop.input-sources sources         # as the user: the system value
gsettings writable org.gnome.desktop.input-sources sources    # locked: false
gio mime application/pdf                                      # as the user, in the GNOME session
```

## License

Apache-2.0

Created with the help of AI
