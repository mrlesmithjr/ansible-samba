# ansible-samba

Ansible role to install/configure Samba

## Build Status

### GitHub Actions

![Molecule Test](https://github.com/mrlesmithjr/ansible-samba/workflows/Molecule%20Test/badge.svg)

### Travis CI

[![Build Status](https://travis-ci.org/mrlesmithjr/ansible-samba.svg?branch=master)](https://travis-ci.org/mrlesmithjr/ansible-samba)



## Requirements

### Collections

This role requires two Ansible collections. Install them before running:

```bash
ansible-galaxy collection install -r requirements.yml
```

`requirements.yml` declares:

```yaml
collections:
  - name: community.general
  - name: ansible.posix
```

`community.general` is required for ZFS dataset management (`community.general.zfs`). `ansible.posix` is required for ACL support (`ansible.posix.acl`). Both are used only when the relevant per-share keys are set (see Share Variables below).

For any required Ansible roles, review:
[requirements.yml](requirements.yml)

## Role Variables

[defaults/main.yml](defaults/main.yml)

### Share Variables

Each entry in `samba_shares` supports the following keys. The standard keys (`name`, `browsable`, `folder_perms`, `group`, `guest_ok`, `owner`, `read_only`, `valid_users`, `writable`) are unchanged. The following keys were added in the Phase 1 backport:

| Key | Type | Description |
|-----|------|-------------|
| `timemachine` | bool | Enables the Time Machine share block in `smb.conf` (`fruit`, `catia`, `streams_xattr` vfs objects, all required `fruit:` parameters). |
| `timemachine_users` | list of strings | When `samba_timemachine_per_user: true`, a subdirectory is created under the share path for each user in this list, owned by that user. |
| `zfs_dataset` | string | ZFS dataset to create before the share directory is created (e.g. `tank/timemachine`). The dataset is created automatically if the key is present; no separate task or variable is needed. |
| `zfs_dataset_properties` | dict | ZFS properties to set on the dataset at creation time (e.g. `compression: lz4`, `atime: "off"`). Passed directly to `community.general.zfs` as `extra_zfs_properties`. |
| `acls` | list of dicts | ACL entries applied to the share directory via `ansible.posix.acl`. Each entry supports `entity`, `etype`, `permissions`, `state` (default `present`), `default`, and `recurse`. ACL application requires `samba_acl_support: true` at the role level. |

### Role-Level Variables for Time Machine

| Variable | Default | Description |
|----------|---------|-------------|
| `samba_acl_support` | `false` | Set to `true` to install the `acl` package and apply `acls` entries from each share. |
| `samba_timemachine_per_user` | `false` | Set to `true` to create per-user subdirectories under Time Machine shares. |

### Time Machine Share Example

```yaml
samba_acl_support: true
samba_timemachine_per_user: true

samba_shares:
  - name: timemachine
    browsable: "yes"
    folder_perms: "0775"
    group: timemachine
    guest_ok: "no"
    owner: root
    read_only: "no"
    writable: "yes"
    timemachine: true
    timemachine_users:
      - alice
      - bob
    zfs_dataset: tank/timemachine
    zfs_dataset_properties:
      compression: lz4
      atime: "off"
    acls:
      - entity: alice
        etype: user
        permissions: rwx
        state: present
      - entity: bob
        etype: user
        permissions: rwx
        state: present
```

When `zfs_dataset` is set, the role creates the ZFS dataset (with any `zfs_dataset_properties`) before creating the share directory. The dataset must reside on a pool that is already imported on the target host.

When `timemachine: true`, the generated `smb.conf` share block includes the full `fruit` VFS stack (`vfs objects = catia fruit streams_xattr`) and all required `fruit:` parameters for macOS Time Machine compatibility, including `fruit:advertise_fullsync = yes` and `fruit:encoding = native`.

## Dependencies

## Example Playbook

[playbook.yml](playbook.yml)

## License

MIT

## Author Information

Larry Smith Jr.

- [@mrlesmithjr](https://twitter.com/mrlesmithjr)
- [mrlesmithjr@gmail.com](mailto:mrlesmithjr@gmail.com)
- [http://everythingshouldbevirtual.com](http://everythingshouldbevirtual.com)

> NOTE: Repo has been created/updated using [https://github.com/mrlesmithjr/cookiecutter-ansible-role](https://github.com/mrlesmithjr/cookiecutter-ansible-role) as a template.
