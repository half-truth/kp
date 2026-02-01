## kp - Interactive Process Killer

A simple yet powerful bash function that allows you to interactively search for and kill processes using `fzf` (fuzzy finder).

### What is this?

`kp` (Kill Process) is a shell function that provides an interactive interface for terminating running processes. Instead of manually finding process IDs and using `kill` commands, you can visually search through your running processes and select which one to terminate.

### Key Features

- **Interactive Search**: Use fuzzy search to find processes by name
- **Process Information**: View CPU usage, memory usage, and full command
- **Graceful Termination**: First attempts graceful termination (SIGTERM)
- **Force Kill Option**: Automatically escalates to force kill (SIGKILL) if needed
- **User-Friendly**: Clear feedback about termination status

### Prerequisites

- **fzf**: Fuzzy finder must be installed
  ```bash
  # Install fzf on macOS
  brew install fzf

  # Install fzf on Ubuntu/Debian
  sudo apt install fzf

  # Install fzf on Fedora
  sudo dnf install fzf
  ```

### Installation

1. Add the `kp` function to your shell configuration file (`~/.bashrc`, `~/.zshrc`, or `~/.bash_profile`):

```bash
function kp() {
    command -v fzf >/dev/null 2>&1 || { echo "kp: fzf not found"; return 127; }
    local pid

    pid=$(ps -Ao pid=,pcpu=,pmem=,command= | \
        fzf --header='[Kill Process] Search Name' --layout=reverse --preview 'echo {}' | \
        awk '{print $1}')

    [[ -n "$pid" ]] || return 0

    kill -TERM -- "$pid" 2>/dev/null || { echo "Failed to terminate PID: $pid"; return 1; }
    sleep 0.2
    if kill -0 -- "$pid" 2>/dev/null; then
        kill -KILL -- "$pid" 2>/dev/null || { echo "Failed to force kill PID: $pid"; return 1; }
        echo "Force killed PID: $pid"
    else
        echo "Terminated PID: $pid"
    fi
}
```

2. Reload your shell configuration:
   ```bash
   source ~/.bashrc  # or ~/.zshrc, depending on your shell
   ```

### How to Use

1. **Run the command**:
   ```bash
   kp
   ```

2. **Search for processes**:
   - A fuzzy search interface will appear showing all running processes
   - Type to filter processes by name or command
   - Use arrow keys to navigate

3. **Select a process**:
   - Press `Enter` to select the highlighted process
   - Press `Esc` or `Ctrl+C` to cancel

4. **Termination process**:
   - The script first sends a SIGTERM (graceful termination)
   - If the process doesn't exit within 0.2 seconds, it sends SIGKILL (force kill)
   - You'll see a confirmation message indicating the result

### Example Workflow

```bash
# 1. Type the command
$ kp

# 2. Search interface appears:
# [Kill Process] Search Name
#   1234  2.5  1.2  /usr/bin/python3 app.py
#   5678  0.5  0.3  /usr/bin/node server.js
#   9012 15.0  5.1  /usr/bin/chrome

# 3. Type "node" to filter, select the Node.js process

# 4. Output:
# Terminated PID: 5678
```

### When to Use This

- **Unresponsive applications**: When GUI applications freeze
- **Background processes**: Servers or daemons that need restarting
- **Resource-heavy processes**: Processes consuming too much CPU/memory
- **Development workflow**: Quickly restart development servers
- **Troubleshooting**: When you need to kill processes but don't remember exact PIDs

### Safety Features

- **No accidental kills**: You must explicitly select a process
- **Graceful first**: Tries normal termination before force killing
- **Clear feedback**: Always tells you what happened
- **No silent failures**: Reports errors if termination fails

### Customization Options

You can modify the function to:
- Change the delay before force kill (adjust `sleep 0.2`)
- Add additional process columns
- Change the fzf theme or colors
- Add confirmation prompts

### Troubleshooting

**"kp: fzf not found"**: Install fzf using your package manager

**Permission denied**: You may need sudo privileges to kill certain system processes

**Process not dying**: Some processes might be in uninterruptible sleep states
