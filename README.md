# ansible-role-gnome

GNOME settings for all users on openSUSE Leap, from the inventory: dconf system defaults and locks. A new
setting is inventory data, no role code.

1. installs `dconf`
2. writes the dconf profile `/etc/dconf/profile/user` (`user-db:user`, `system-db:local`): every user reads
   their own settings first, then the system db `local`
3. writes `/etc/dconf/db/local.d/50-ansible` from `settings` (checked with `dconf compile` before it replaces
   the file: an invalid value stops the run and names key and value)
4. writes `/etc/dconf/db/local.d/locks/50-ansible` from `locks`
5. on changes: `dconf update` (compiles `/etc/dconf/db/local`)

Without `settings` / `locks` the role removes its own files again (desired state); the profile stays (harmless).
Users see changes after logging in again.

**Default or lock:** a setting alone is only the default - a user who already stored their own value (GNOME
writes some at first login, e.g. the keyboard layouts) keeps it. A locked key always has the system value, also
for those users, and users can't change it.

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
```

Find keys and values: `gsettings list-recursively org.gnome.desktop.input-sources`, or change the setting in
GNOME Settings and watch `dconf watch /`.

## Check

```bash
cat /etc/dconf/db/local.d/50-ansible /etc/dconf/db/local.d/locks/50-ansible
gsettings get org.gnome.desktop.input-sources sources         # as the user: the system value
gsettings writable org.gnome.desktop.input-sources sources    # locked: false
```

## License

Apache-2.0

Created with the help of AI
