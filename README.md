# ansible-samba

An [Ansible](https://www.ansible.com) role to install and configure [Samba](https://www.samba.org).

## Ansible Galaxy

```bash
ansible-galaxy install mrlesmithjr.samba
```

## Requirements

Run with `become: true`. This role requires two collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

`requirements.yml` declares `community.general` (ZFS dataset management) and `ansible.posix` (ACL support). Both activate only when the relevant per-share keys are set.

## Role Variables

See [defaults/main.yml](defaults/main.yml) for the full variable reference.

### Share variables

Each entry in `samba_shares` accepts these keys in addition to the standard ones (`name`, `browsable`, `guest_ok`, `owner`, `group`, `read_only`, `writable`, `valid_users`, `folder_perms`):

| Key | Type | Description |
|-----|------|-------------|
| `timemachine` | bool | Enables the Time Machine share block (`fruit`, `catia`, `streams_xattr` vfs objects and all required `fruit:` parameters) |
| `timemachine_users` | list | When `samba_timemachine_per_user: true`, creates a per-user subdirectory under the share path for each name in this list |
| `zfs_dataset` | string | ZFS dataset to create before the share directory (e.g. `tank/timemachine`). Created automatically when set. |
| `zfs_dataset_properties` | dict | ZFS properties to apply at dataset creation time (e.g. `compression: lz4`) |
| `acls` | list | ACL entries applied via `ansible.posix.acl`. Each entry takes `entity`, `etype`, `permissions`, `state`, `default`, `recurse`. Requires `samba_acl_support: true`. |

### Role-level variables for Time Machine

| Variable | Default | Description |
|----------|---------|-------------|
| `samba_acl_support` | `false` | Install the `acl` package and apply `acls` entries |
| `samba_timemachine_per_user` | `false` | Create per-user subdirectories under Time Machine shares |

## Example Playbook

Standard share:

```yaml
- hosts: samba_servers
  become: true
  roles:
    - role: mrlesmithjr.samba
      vars:
        samba_shares:
          - name: data
            browsable: "yes"
            guest_ok: "no"
            owner: root
            group: smbusers
            read_only: "no"
            writable: "yes"
```

Time Machine share with ZFS and ACLs:

```yaml
- hosts: samba_servers
  become: true
  roles:
    - role: mrlesmithjr.samba
      vars:
        samba_acl_support: true
        samba_timemachine_per_user: true
        samba_shares:
          - name: timemachine
            browsable: "yes"
            guest_ok: "no"
            owner: root
            group: timemachine
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
              - entity: bob
                etype: user
                permissions: rwx
```

When `timemachine: true`, the generated `smb.conf` block includes `vfs objects = catia fruit streams_xattr` and the required `fruit:` parameters including `fruit:advertise_fullsync = yes` and `fruit:encoding = native`.

## Testing

```bash
pip install molecule molecule-docker
molecule test
```

## License

MIT

## Author

Larry Smith Jr. — [everythingshouldbevirtual.com](http://everythingshouldbevirtual.com) · [mrlesmithjr@gmail.com](mailto:mrlesmithjr@gmail.com)
