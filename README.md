# isolate

A secure wrapper script for running [opencode](https://opencode.ai) in an isolated environment using [bubblewrap](https://github.com/containers/bubblewrap).

## Overview

The `isolate` script provides a sandboxed environment for opencode, restricting file system access to only the current repository directory. This helps prevent unintended modifications to your system while using AI-powered code assistance.

## Features

- **Secure isolation**: Uses bubblewrap to create a minimal sandbox
- **Repository-only access**: Only allows access to the current repository directory
- **Network access**: Maintains network connectivity for AI functionality
- **Custom mounts**: Supports additional read-only and read-write directory mounts
- **Transparent operation**: Passes all opencode arguments through seamlessly

## Dependencies

- **bubblewrap**: For sandboxing functionality
- **opencode**: The AI-powered coding assistant

## Installation

### Quick Install

1. Download the script and make it executable:
   ```bash
   curl -O https://raw.githubusercontent.com/your-repo/isolate/main/isolate
   chmod +x isolate
   ```

2. Move to a directory in your PATH:
   ```bash
   sudo mv isolate /usr/local/bin/
   ```

### Manual Install

1. Clone this repository or download the `isolate` script
2. Make the script executable: `chmod +x isolate`
3. Place it somewhere in your PATH (e.g., `/usr/local/bin/`, `~/.local/bin/`)

### Install Dependencies

**Ubuntu/Debian:**
```bash
sudo apt install bubblewrap
```

**Fedora:**
```bash
sudo dnf install bubblewrap
```

**Arch Linux:**
```bash
sudo pacman -S bubblewrap
```

## Usage

### Basic Usage

Run opencode in the current repository directory:
```bash
isolate
```

All standard opencode arguments are supported:
```bash
isolate --help
isolate "Add error handling to the main function"
```

### Additional Mount Options

Mount additional directories as read-only:
```bash
isolate --ro /path/to/docs /usr/share/templates
```

Mount additional directories as read-write:
```bash
isolate --rw /tmp/workspace /home/user/shared
```

Combine multiple mount types:
```bash
isolate --ro /usr/share/docs /etc/config --rw /tmp/build /var/output "Build the project"
```

## How It Works

The script creates a bubblewrap sandbox that:

1. **Mounts the repository** at `/workspace` (read-write)
2. **Provides essential system access** (read-only):
   - System libraries (`/usr`, `/lib`, `/bin`, etc.)
   - Network configuration files
   - SSL certificates
   - Package manager tools (uv, mise, homebrew)
3. **Preserves opencode functionality**:
   - Mounts opencode binary and dependencies
   - Maintains access to opencode's config directory
   - Preserves network access for AI features
4. **Isolates everything else**: No access to home directory or other filesystem locations

## Security Benefits

- **Prevents accidental system modifications**: AI can only modify files in the repository
- **Protects sensitive data**: No access to home directory, SSH keys, or other personal files
- **Limits blast radius**: Even if something goes wrong, damage is contained to the repository
- **Transparent operation**: Works exactly like regular opencode but safer

## Environment Variables

- `TRACE=1`: Enable debug tracing to see all executed commands

## Troubleshooting

### "bubblewrap (bwrap) is not installed"
Install bubblewrap using your system's package manager (see Installation section).

### "opencode is not installed or not in PATH"
Ensure opencode is properly installed and accessible. Test with `opencode --help`.

### Permission denied errors
Make sure the `isolate` script is executable: `chmod +x isolate`

### Extra mount path does not exist
Verify that paths specified with `--ro` or `--rw` exist and are accessible.

## Contributing

Feel free to submit issues and enhancement requests. The script is designed to be simple and focused on security through isolation.

## License

This project is released under the same license as your preferred open source license.