# isolate

A secure wrapper script for running [opencode](https://opencode.ai) in an
isolated environment using [bubblewrap](https://github.com/containers/bubblewrap).

## Overview

The `isolate` script provides a sandboxed environment for opencode, restricting
write access by default to the current directory you start it in and the
opencode config dir. This helps prevent unintended modifications to your system
while using AI-powered code assistance.

The main purpose is to restrict write access to just a few directories.  I
originally started out trying to restrict a lot of its read access, but
eventually caved in and gave it full read access to my home directory so it can
see all the dotfiles that live there, like `.gitignore`.  This way it will stop
checking random crap into my repos every time because it can't see my global git
ignore file.

## Features

- **Secure isolation**: Uses bubblewrap to create a minimal sandbox
- **Limited write access**: Only allows write access to the current directory (you can specify additional directories)
- **Directory hiding**: Hide sensitive directories with tmpfs overlays
- **Network access**: Maintains network connectivity for AI functionality
- **Custom mounts**: Supports additional read-only and read-write directory mounts
- **Docker integration**: Optional Docker socket access for containerization tasks
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
isolate --ro /path/to/docs /usr/share/templates -- run "Build docs"
```

Mount additional directories as read-write:
```bash
isolate --rw /tmp/workspace /home/user/shared -- "Process files"
```

Hide directories with tmpfs overlays:
```bash
isolate --hide /home/user/.secrets /tmp/sensitive -- auth
```

Enable Docker socket access for containerization tasks:
```bash
isolate --docker run "Help me containerize this application"
```

Combine multiple mount types:
```bash
isolate --ro /usr/share/docs --rw /tmp/build --hide /home/user/.env -- run "Build project"
```

Use Docker with additional mounts:
```bash
isolate --docker --rw /tmp/build -- run "Create a Dockerfile and test the build"
```

**Note**: Use `--` to separate mount options from opencode arguments (recommended when using mount options with opencode commands).

### Docker Integration

The `--docker` flag enables access to the Docker daemon socket, allowing opencode to execute Docker commands within the isolated environment. This is useful for containerization tasks, Docker-based development workflows, and container management.

**Requirements for Docker access:**
- Docker must be installed and running on the host system
- The current user must have permission to access the Docker socket (usually by being in the `docker` group)
- The Docker socket must exist at `/var/run/docker.sock`

**Docker usage examples:**
```bash
# Basic Docker access
isolate --docker

# Ask opencode to create and test a Dockerfile
isolate --docker run "Create a Dockerfile for this Node.js app and test the build"

# Docker with additional workspace access
isolate --docker --rw /tmp/docker-builds run "Build and push container images"

# Multiple Docker operations
isolate --docker run "Create docker-compose.yml, build images, and show running containers"
```

**Security considerations:**
- Docker access is opt-in only - it must be explicitly enabled with `--docker`
- The Docker socket provides significant system access, so only use when needed
- Docker commands run with the same user permissions as the host system
- Container builds and runs will have access to the Docker daemon's capabilities

## How It Works

The script creates a bubblewrap sandbox that:

1. **Mounts the current directory** at `/workspace` (read-write)
2. **Provides essential system access** (read-only):
   - System libraries (`/usr`, `/lib`, `/bin`, etc.)
   - Network configuration files
   - SSL certificates
   - Package manager tools (uv, mise, homebrew)
   - Your home directory (read-only, for access to dotfiles)
3. **Optional Docker access** (when `--docker` is used):
   - Mounts Docker socket at `/var/run/docker.sock`
   - Provides access to Docker binary and related tools
4. **Preserves opencode functionality**:
   - Mounts opencode binary and dependencies
   - Maintains access to opencode's config directory
   - Preserves network access for AI features
5. **Hides sensitive directories** with tmpfs overlays:
   - `~/.ssh` (SSH keys)
   - `~/.aws` (AWS credentials)
   - `~/.gnupg` (GPG keys)
   - Additional directories can be hidden with `--hide`

## Directory Hiding with tmpfs

The script automatically hides sensitive directories using tmpfs overlays, which means these directories appear empty inside the sandbox even though they exist on your system. This prevents AI from accidentally reading or modifying sensitive files.

### Default Hidden Directories

- `~/.ssh` - SSH private keys and configuration
- `~/.aws` - AWS credentials and configuration
- `~/.gnupg` - GPG keys and configuration
- `/tmp`, `/var/tmp`, `/run` - Temporary directories (replaced with clean tmpfs)

### Custom Hidden Directories

Use the `--hide` option to hide additional directories:

```bash
isolate --hide /path/to/sensitive/dir -- run "Process data safely"
```

## Security Benefits

- **Prevents accidental system modifications**: AI can only modify files in the repository and explicitly mounted directories
- **Protects sensitive data**: Uses tmpfs overlays to hide sensitive directories like SSH keys, AWS credentials, and GPG keys
- **Limited home directory access**: Read-only access to home directory for dotfiles, but sensitive subdirectories are hidden
- **Limits blast radius**: Even if something goes wrong, damage is contained to allowed directories
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
Verify that paths specified with `--ro`, `--rw`, or `--hide` exist and are accessible.

### Docker socket access issues
If Docker commands fail with the `--docker` flag:
- Ensure Docker is running: `sudo systemctl status docker`
- Check if you're in the docker group: `groups | grep docker`
- Add yourself to docker group if needed: `sudo usermod -aG docker $USER` (requires logout/login)
- Verify socket exists: `ls -la /var/run/docker.sock`

### Using mount options with opencode commands
When using mount options with opencode commands, use the `--` delimiter:
```bash
# Correct
isolate --ro /path -- run "command"

# Incorrect - will try to mount "run" as a directory
isolate --ro /path run "command"
```

## Contributing

Feel free to submit issues and enhancement requests. The script is designed to be simple and focused on security through isolation.

## License

This project is released under the same license as your preferred open source license.