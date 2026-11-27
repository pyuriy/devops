# Basics of tmux

`tmux` is included in the default **BaseOS / AppStream** repositories on Rocky Linux (versions 8, 9, and 10), so no extra repositories like EPEL are required.

### Installation Steps

1. **Install `tmux` using `dnf`:**
```bash
sudo dnf install -y tmux

```


*(Verification: Command finishes with `Complete!` and no dependency errors.)*
2. **Verify the installation:**
```bash
tmux -V

```


*(Verification: Outputs the installed version, such as `tmux 3.2a` on Rocky 9 or `tmux 2.7` on Rocky 8.)*

---

### Offline / Minimal Repository Fallback

If `dnf` fails because repository metadata is out of date or BaseOS/AppStream is disabled:

```bash
sudo dnf clean all
sudo dnf makecache
sudo dnf install -y tmux

```

## tmux cheatsheet 

---

### Sessions

| Command | Action |
| --- | --- |
| `tmux` | Start a new unnamed session |
| `tmux new -s <name>` | Start a new session named `<name>` |
| `tmux ls` | List all running sessions |
| `tmux attach -t <name>` | Attach to an existing session (or `tmux a -t <name>`) |
| `Prefix` then `d` | Detach from the current session |
| `tmux kill-session -t <name>` | Kill a specific session |
| `tmux kill-server` | Terminate all sessions and background tmux server |

---

### Windows (Tabs)

| Key Combination | Action |
| --- | --- |
| `Prefix` then `c` | Create a new window |
| `Prefix` then `,` | Rename the current window |
| `Prefix` then `n` | Go to next window |
| `Prefix` then `p` | Go to previous window |
| `Prefix` then `0–9` | Jump directly to window by number |
| `Prefix` then `w` | Show an interactive menu of all windows and sessions |
| `Prefix` then `&` | Kill the current window |

---

### Panes (Splits)

| Key Combination | Action |
| --- | --- |
| `Prefix` then `"` | Split pane horizontally (top / bottom) |
| `Prefix` then `%` | Split pane vertically (left / right) |
| `Prefix` then `Arrow Key` | Move cursor to adjacent pane |
| `Prefix` then `o` | Rotate to next pane |
| `Prefix` then `z` | Toggle zoom (maximize/restore active pane) |
| `Prefix` then `x` | Close/kill current pane |
| `Prefix` then `Space` | Cycle through preset pane layouts |
| `Prefix` then `Ctrl + Arrow` | Resize active pane by 1 cell |

---

### Navigation & Copy Mode

| Key Combination | Action |
| --- | --- |
| `Prefix` then `[` | Enter copy / scrollback mode (use arrow keys or PageUp/PageDown to scroll) |
| `q` | Exit copy mode |
| `Prefix` then `]` | Paste buffer content into active pane |
| `Prefix` then `:` | Open tmux command prompt |
| `Prefix` then `?` | Show full list of keybindings (press `q` to exit) |


### macOS & iTerm2 Pitfalls

On macOS inside iTerm2, the key combination works almost identically to Linux—**you still use the physical `Control` (`⌃`) key, NOT the `Command` (`⌘`) key**.

Because macOS uses `Command (⌘)` for system and app shortcuts (like `⌘ + C` to copy or `⌘ + W` to close a tab) and reserves `Control (⌃)` for terminal signals, iTerm2 passes `Control + b` directly through to tmux as the ASCII character `STX` (hex `0x02`).

---

### How to Execute It Step-by-Step

Using splitting the screen vertically (`Prefix` then `%`) as an example:

1. **Press and hold** the **`control` (`⌃`)** key on your Mac keyboard (located at the bottom-left corner, next to `fn` or `option`).
2. **Tap** the **`b`** key while still holding `control`.
3. **Release both keys completely.** (Nothing will visibly happen on your screen yet—tmux is now silently waiting for your next keystroke).
4. **Press** the command key—for example, `Shift + 5` (to type `%`).
* *Verification:* Your iTerm2 window immediately splits into left and right panes.



---

### Common macOS & iTerm2 Pitfalls

| Issue | Cause & Fix |
| --- | --- |
| **Using `Command (⌘) + b` instead of `Control (⌃) + b**` | `Command + b` is intercepted by macOS/iTerm2 (or ignored) and never reaches tmux. Always use **`control` (`⌃`)**. |
| **Arrow keys after `Prefix` switch macOS Spaces instead of tmux panes** | By default, macOS maps `Control + Left/Right Arrow` to Mission Control ("Move left/right a space"). If you hold `control` too long when pressing an arrow key, macOS intercepts it. Make sure you **release `control` after pressing `b**`, or disable those shortcuts in **System Settings → Keyboard → Keyboard Shortcuts → Mission Control**. |
| **`Option` key shortcuts (like resizing panes) don't work** | If you use `Option` (`⌥`) in custom tmux bindings, open **iTerm2 Settings (`⌘ + ,`) → Profiles → Keys → General** and set **Left Option key** to **Esc+**. |
