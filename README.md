# push-load – Docker Image Distribution with Ansible

Deploy Docker images to remote servers: pull, save as tarballs, copy, and load.

## Quick Start

**1. Create config file:**
```bash
cp config.yml.example config.yml
```

**2. Edit `config.yml`:**
```yaml
target_hosts:
  - name: server1
    ip: 192.168.1.100

mode: "images"  # or "tarballs" or "mixed"

docker_images:
  - nginx:latest
  - redis:7-alpine
```

**3. Run:**
```bash
ansible-playbook playbooks/push-load.yml -e "@config.yml"
```

## Operation Modes

### `images` - Pull and deploy from Docker Hub
```yaml
mode: "images"
docker_images:
  - nginx:latest
```

### `tarballs` - Use existing .tar files
```yaml
mode: "tarballs"
tarballs_dir: "/path/to/tarballs"
```

### `mixed` - Combine tarballs and images
```yaml
mode: "mixed"
tarballs_dir: "/path/to/tarballs"
docker_images:
  - nginx:latest
```

## Config File Options

```yaml
target_hosts:
  - name: server1
    ip: 192.168.1.100

mode: "images"              # Required: "images", "tarballs", or "mixed"
tarballs_dir: "/path/to"    # For tarballs/mixed mode
docker_images:              # For images/mixed mode
  - nginx:latest
skip_pull: false            # Skip pulling (images must exist locally)
remote_user: "ubuntu"       # SSH user for remote hosts
```

## Prerequisites

- Docker on controller and target servers
- Ansible installed
- SSH access to targets (default user: `ubuntu`)

## Troubleshooting

**SSH connection failed:**
```bash
ssh ubuntu@SERVER_IP  # Test manually
```

**No tarballs found:**
- Check `tarballs_dir` path is correct
- Verify `.tar` files exist in directory

**Docker permission denied:**
```bash
# On target server:
sudo usermod -aG docker ubuntu
```

## Examples

**Deploy from current directory:**
```yaml
mode: "tarballs"
tarballs_dir: "."
target_hosts:
  - name: server1
    ip: 192.168.1.100
```

**Quick CLI usage (legacy):**
```bash
ansible-playbook playbooks/push-load.yml -e "docker_images=nginx:latest"
```

---

**Note:** Tarballs are saved to `~/.docker-push-load/` by default.
