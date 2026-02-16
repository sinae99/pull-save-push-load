# push and load

An Ansible playbook to **pull** Docker images, **save** them as tarballs, **copy** to remote hosts, and **load** them into Docker on target machines.

## start

**1. Configure your target hosts**  
Edit `inventory/hosts.ini` and add your target server IPs:

```ini
[target_hosts]
vm1 ansible_host=192.168.1.100
vm2 ansible_host=192.168.1.101
vm3 ansible_host=192.168.1.102
```

**2. Run with a single image:**
```bash
ansible-playbook playbooks/push-load.yml -e "docker_images=busybox:latest"
```

**3. Run with multiple images:**
```bash
ansible-playbook playbooks/push-load.yml -e "docker_images='busybox:latest redis:7.2-alpine nginx:latest'"
```

playbook will automatically:
- Check if images exist locally (skips pull if already present)
- Pull missing images from Docker Hub/registry
- Save images as tarballs in `~/.docker-push-load/` (your home directory)
- Copy tarballs to all target hosts
- Load images into Docker on remote hosts
- Clean up temporary files

## requisites

- **Controller machine** (where you run the playbook):
  - Docker installed and running
  - Ansible installed
  - SSH access to target hosts

- **Target hosts**:
  - Docker installed
  - SSH access with `ubuntu` user (or configure `push_load_remote_user`)
  - User must have sudo privileges for Docker commands

## Usage Examples

### Example 1: Single Image
Pull, save, and deploy `busybox:latest` to all hosts:
```bash
ansible-playbook playbooks/push-load.yml -e "docker_images=busybox:latest"
```

### Example 2: Multiple Images (Space-Separated)
Deploy multiple images at once:
```bash
ansible-playbook playbooks/push-load.yml -e "docker_images='busybox:latest redis:7.2-alpine nginx:latest'"
```

### Example 3: Use Images from Inventory Config
If you've configured images in `inventory/group_vars/all.yml`, just run:
```bash
ansible-playbook playbooks/push-load.yml
```

### Example 4: Skip Pull (Images Already Local)
If images are already on your controller machine:
```bash
ansible-playbook playbooks/push-load.yml -e "docker_images=busybox:latest" -e "push_load_skip_pull=true"
```

### Example 5: Use Existing Tarballs
If you already have `.tar` files saved:
```bash
ansible-playbook playbooks/push-load.yml -e "push_load_tarballs_dir=/path/to/dir/with/tars"
```

### Example 6: Deploy to Specific Hosts Only
Limit deployment to specific hosts:
```bash
ansible-playbook playbooks/push-load.yml -e "docker_images=busybox:latest" -l vm1,vm2
```

## Configuration

### Inventory File (`inventory/hosts.ini`)

Define your target hosts:
```ini
[target_hosts]
vm1 ansible_host=192.168.1.100
vm2 ansible_host=192.168.1.101
vm3 ansible_host=192.168.1.102
```

### Group Variables (`inventory/group_vars/all.yml`)

Set default images and options:
```yaml
# Default images to deploy
docker_images:
  - citusdata/pg_auto_failover:latest
  - redis:7.2-alpine
  - mongo:7

# Skip pulling images (they must exist locally)
push_load_skip_pull: false

# Directory for tarballs (defaults to ~/.docker-push-load/)
push_load_tarball_dir: "/tmp/docker-push-load"

# SSH user for remote hosts
push_load_remote_user: "ubuntu"
```

## Variables Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `docker_images` | (from `all.yml`) | List of Docker images to pull/save/load. Can be a single string, space-separated string, or YAML list. |
| `push_load_skip_pull` | `false` | If `true`, skip pulling images (they must already exist on controller). |
| `push_load_tarball_dir` | `~/.docker-push-load` | Directory on controller for saved tarballs. Automatically uses home directory if `/tmp` is not writable. |
| `push_load_tarballs_dir` | (none) | If set, use existing `.tar` files from this directory (skips pull and save steps). |
| `push_load_remote_user` | `ubuntu` | SSH user for connecting to target hosts. |

## How It Works

1. **Image Normalization**: Converts input (string/list) to a consistent list format
2. **Existence Check**: Checks which images already exist locally using `docker image inspect`
3. **Pull**: Only pulls images that don't exist locally (saves time and bandwidth)
4. **Save**: Saves all images as `.tar` files in the tarball directory
5. **Copy to Targets**: Transfers tarballs to all target hosts via SSH
6. **Load**: Loads images into Docker on each remote host
7. **Cleanup**: Removes temporary staging directories on remote hosts

## File Structure

```
push-load/
├── ansible.cfg              # Ansible configuration
├── inventory/
│   ├── hosts.ini            # Target host definitions
│   └── group_vars/
│       └── all.yml          # Default variables
├── playbooks/
│   └── push-load.yml        # Main playbook
└── README.md                # This file
```
