
# sway_config
 
My desktop config for **EndeavourOS / Arch** — sway (Wayland) with a retro-futuristic
theme, based on [diinki's rice](https://github.com/diinki) (MIT).
 
![wallpaper colors](https://img.shields.io/badge/palette-%23141216%20%23d8cab8%20%23AC82E9-AC82E9)
 
| | |
|---|---|
| WM | sway |
| Bar | waybar |
| Launcher | wofi |
| Terminal | kitty |
| Files | nautilus |
| Editor | vim / neovim |
 
---
 
## Repo layout
 
```
sway/config                 → ~/.config/sway/config
waybar/config               → ~/.config/waybar/config
waybar/style.css            → ~/.config/waybar/style.css
wofi/config                 → ~/.config/wofi/config
wofi/style.css              → ~/.config/wofi/style.css
kitty/kitty.conf            → ~/.config/kitty/kitty.conf
kitty/diinki_retro.conf     → ~/.config/kitty/diinki_retro.conf
nvim/init.vim               → ~/.config/nvim/init.vim
vimrc                       → ~/.vimrc
gtk-3.0/                    → ~/.config/gtk-3.0/
gtk-4.0/                    → ~/.config/gtk-4.0/
```
 
`gtk-3.0/bookmarks` is the Nautilus sidebar. It hardcodes `/home/iksrigz`, so edit it
if the username differs.
 
`gtk-4.0/` holds the libadwaita CSS override — see [GTK theme](#gtk-theme-dark-apps).
 
---
 
## Install
 
### 1. Packages
 
```bash
# core session
sudo pacman -S --needed sway swaybg swayidle swaylock waybar wofi kitty
 
# audio
sudo pacman -S --needed pipewire pipewire-pulse wireplumber pavucontrol
 
# network + bluetooth
sudo pacman -S --needed networkmanager network-manager-applet nm-connection-editor \
                        bluez bluez-utils blueman
 
# screenshots + clipboard
sudo pacman -S --needed grim slurp swappy wl-clipboard
 
# hardware keys
sudo pacman -S --needed brightnessctl playerctl
 
# apps
sudo pacman -S --needed nautilus imv mpv geany vim neovim
 
# fonts
sudo pacman -S --needed ttf-nerd-fonts-symbols
```
 
**Maple Mono** (used by sway, waybar, wofi and kitty) is not in the official repos:
 
```bash
yay -S ttf-maple
```
 
`CaskaydiaCove Nerd Font` is referenced by the exit dialog in `sway/config` — either
install it (`yay -S ttf-cascadia-code-nerd`) or edit that `--font=` argument.
 
### 2. Link the configs
 
```bash
git clone <this-repo> ~/.config/rice
cd ~/.config/rice
 
mkdir -p ~/.config/{sway,waybar,wofi,kitty,nvim}
ln -sf "$PWD"/sway/config              ~/.config/sway/config
ln -sf "$PWD"/waybar/config            ~/.config/waybar/config
ln -sf "$PWD"/waybar/style.css         ~/.config/waybar/style.css
ln -sf "$PWD"/wofi/config              ~/.config/wofi/config
ln -sf "$PWD"/wofi/style.css           ~/.config/wofi/style.css
ln -sf "$PWD"/kitty/kitty.conf         ~/.config/kitty/kitty.conf
ln -sf "$PWD"/kitty/diinki_retro.conf  ~/.config/kitty/diinki_retro.conf
ln -sf "$PWD"/nvim/init.vim            ~/.config/nvim/init.vim
ln -sf "$PWD"/vimrc                    ~/.vimrc
 
# GTK — install the theme first, see below
ln -sfn "$PWD"/gtk-3.0                 ~/.config/gtk-3.0
ln -sfn "$PWD"/gtk-4.0                 ~/.config/gtk-4.0
```
 
`-n` on the last two so they don't nest inside an existing directory.
 
### 3. Edit what's machine-specific
 
In `sway/config`:
 
- **Monitors** — `swaymsg -t get_outputs` to get your names/modes, then edit the
  `output` lines. Lines for outputs that aren't connected are ignored, so one config
  can cover docked and undocked.
- **Wallpaper** — the `output * bg` path.
- **Keyboard layout** — `xkb_layout "us,se"`, toggle is Alt+Shift.
---
 
## Services and one-time setup
 
### Bluetooth
 
Not enabled by default on EndeavourOS:
 
```bash
sudo systemctl enable --now bluetooth
```
 
So the adapter powers on at boot, in `/etc/bluetooth/main.conf` under `[Policy]`:
 
```
AutoEnable=true
```
 
`blueman-applet` is started from `sway/config`. **The manager won't work without it** —
it owns the D-Bus service and handles pairing dialogs.
 
### Network
 
```bash
sudo systemctl enable --now NetworkManager
```
 
`nm-applet` is started from `sway/config`; the waybar network module opens
`nm-connection-editor` on click.
 
### Screenshots
 
```bash
mkdir -p ~/Pictures/Screenshots
```
 
| Key | Action |
|---|---|
| `Print` | whole screen → `~/Pictures/Screenshots/` + clipboard |
| `$mod+Shift+s` | select region → opens swappy to annotate and save |
 
### GTK theme (dark apps)
 
```bash
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
gsettings set org.gnome.desktop.interface gtk-theme 'diinki-retro-dark'
```
 
The theme folder goes in `~/.themes/`:
 
```bash
mkdir -p ~/.themes
cp -r /path/to/diinki-retro-dark ~/.themes/
```
 
That covers **GTK3** only. GTK4/libadwaita apps (Nautilus) ignore both `GTK_THEME` and
`gtk-theme` — libadwaita has no theme system, and `~/.config/gtk-4.0/gtk.css` is the
only hook. That's what the tracked `gtk-4.0/` directory is for.
 
Its contents are symlinks into `~/.themes/diinki-retro-dark/gtk-4.0/`, so **install the
theme before linking `gtk-4.0/`** or they dangle. Verify after:
 
```bash
ls -l ~/.config/gtk-4.0/     # no red / "broken" entries
nautilus -q                  # restart to pick it up
```
 
If you'd rather the repo be self-contained, replace the symlinks with the real files
(`cp -rL`) and the theme install step above becomes GTK3-only.
 
### Default applications
 
Without a desktop environment nothing registers these, so files fall through to the
browser:
 
```bash
xdg-mime default imv.desktop image/png image/jpeg image/webp image/gif
xdg-mime default org.gnome.Evince.desktop application/pdf
 
xdg-mime query default image/png   # verify
```
 
### XDG environment variables
 
A bare sway session leaves `XDG_DATA_HOME` / `XDG_DATA_DIRS` unset, which means
`.desktop` files in `~/.local/share/applications` may not appear in wofi:
 
```bash
mkdir -p ~/.config/environment.d
printf 'XDG_DATA_HOME=%s/.local/share\nXDG_DATA_DIRS=/usr/local/share:/usr/share\n' \
  "$HOME" > ~/.config/environment.d/xdg.conf
```
 
Takes effect at next login.
 
---
 
## Keybinds
 
`$mod` = Super.
 
| Key | Action |
|---|---|
| `$mod+Return` | terminal (kitty) |
| `$mod+d` | app launcher (wofi) |
| `$mod+e` | file manager |
| `$mod+q` | close window |
| `$mod+f` | fullscreen |
| `$mod+h/j/k/l` | move focus |
| `$mod+Shift+h/j/k/l` | move window |
| `$mod+1..0` | switch workspace |
| `$mod+Shift+1..0` | move window to workspace |
| `$mod+a` | focus parent container |
| `$mod+Shift+space` | toggle floating |
| `$mod+Shift+c` | reload config |
| `$mod+Shift+e` | exit sway (with confirmation) |
| `Print` / `$mod+Shift+s` | screenshots (above) |
 
Media and brightness keys are bound with `--locked`, so they work on the lock screen.
 
---
 
## Adding your own apps to the launcher
 
wofi reads `.desktop` files from `~/.local/share/applications/`. For anything not
installed by pacman (AppImages, extracted debs):
 
```ini
[Desktop Entry]
Type=Application
Name=My App
Exec=my-app
Icon=applications-other
Terminal=false
Categories=Utility;
```
 
Two gotchas:
 
- With `show_all=false` in the wofi config, entries whose `Exec` isn't on `$PATH` are
  hidden. Put a wrapper or symlink in `~/.local/bin` rather than using an absolute path.
- An AppImage that self-updates changes its filename and breaks a hardcoded `Exec`.
  A wrapper avoids that:
```bash
printf '#!/bin/sh\nexec "$(ls -t "$HOME"/Apps/My-App-*.AppImage | head -1)" "$@"\n' \
  > ~/.local/bin/my-app
chmod +x ~/.local/bin/my-app
```
 
---
 
## Notes / gotchas
 
- **Gaps set with `swaymsg` override the config** for existing workspaces, and `reload`
  does not reset them. After editing gaps in the config, log out and back in — sway has
  no in-place restart.
- **Inner gaps only show between windows.** With one window open, changing
  `gaps inner` looks like nothing happened.
- **`pkill speech-dispatcher` silently does nothing** (process names are capped at 15
  characters). Use `pkill -f`.
- **waybar** is started via the `bar { swaybar_command waybar }` block, so
  `$mod+Shift+c` restarts it along with sway.
---
 
## Credits
 
Theme, colors and the original config: [diinki](https://github.com/diinki) — MIT License.
 

