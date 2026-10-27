By default, all tmux commands start with the prefix key: **`Ctrl + b`** (written below as `Prefix`). Press `Ctrl + b`, release both keys, and then press the command key.

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
