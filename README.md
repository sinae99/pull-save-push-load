# push-load 

a playbook to **pull** , **save** images to `.tar`, **copy** them to target hosts, and **load** into Docker there.

## Prerequisites

- Docker on the **controller** (the machine where you run `ansible-playbook`) for pull/save.
- Docker on all **target hosts**; SSH (e.g. `ubuntu`) with sudo for `docker load`.
- Edit `inventory/hosts.ini` with your target IPs.

## Usage

**1. Full run (pull → save → copy → load)**  
Uses `docker_images` from `inventory/group_vars/all.yml`:

```bash
cd push-load
ansible-playbook playbooks/push-load.yml
```

**2. Skip pull (images already on controller)**  
Only save → copy → load:

```bash
ansible-playbook playbooks/push-load.yml -e "push_load_skip_pull=true"
```

**3. Custom image list**  
Override from the CLI:

```bash
ansible-playbook playbooks/push-load.yml -e "docker_images=citusdata/pg_auto_failover:latest redis:7.2-alpine mongo:7"
```

**4. Only push existing tarballs (no pull, no save)**  
You already have `.tar` files in a directory:

```bash
ansible-playbook playbooks/push-load.yml -e "push_load_tarballs_dir=/path/to/dir/with/tars"
```

**5. Limit to specific hosts**

```bash
ansible-playbook playbooks/push-load.yml -l vm1,vm2
```

## Example Variables (inventory/group_vars/all.yml or -e)

| Variable | Value | Description |
|----------|---------|-------------|
| `docker_images` | pg_auto_failover, redis, mongo | List of image names to pull/save/load. |
| `push_load_skip_pull` | `false` | If `true`, do not pull; images must exist on controller. |
| `push_load_tarball_dir` | `/tmp/docker-push-load` | Dir on controller for saved tars; same base used on remotes for staging. |
| `push_load_tarballs_dir` | (none) | If set, use only existing `.tar` in this dir (no pull, no save). |

## Layout

- `inventory/hosts.ini` – target hosts (your 3 IPs or more).
- `inventory/group_vars/all.yml` – default `docker_images` and options.
- `playbooks/push-load.yml` – single playbook.

This folder is separate from `ansible-old` and `final`; use it whenever you need to ship images to the same (or different) hosts.
