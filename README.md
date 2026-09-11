# Ansible Role: fail2ban

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-fail2ban)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-fail2ban)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-fail2ban)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-fail2ban/dev.yml?branch=dev&event=push&label=dev)](https://github.com/jomrr/ansible-role-fail2ban/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-fail2ban/main.yml?branch=main&event=push&label=main)](https://github.com/jomrr/ansible-role-fail2ban/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for setting up fail2ban.

## Purpose

Install fail2ban, manage daemon settings and selected jail, filter, and action
options, and enable and start the service. Active protection depends on the
enabled jails, matching log sources, and configured ban actions.

## Scope

### Managed

- The fail2ban package, daemon configuration, and service state.
- Selected INI options in jail.local and named action.d, filter.d, and jail.d
  overrides.

### Not Managed

- Package repositories, application logs, and firewall tooling required by
  selected actions.
- Automatic deletion of files or options omitted from inventory.

## Requirements

- Gather Ansible facts, including the default IPv4 route for the default
  ignoreip setting.
- The fail2ban package must be available from the configured package
  repositories.
- Enabled jails require existing log sources and the tools used by their
  actions.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
```

## Role Variables

### `fail2ban_dbfile`

Type: `str`. Required: `false`.

Persistent ban database path; use 'None' to disable persistence.

Default:

```yaml
fail2ban_dbfile: /var/lib/fail2ban/fail2ban.sqlite3
```

### `fail2ban_dbmaxmatches`

Type: `int`. Required: `false`.

Maximum number of matched log entries stored per ban in the database.

Default:

```yaml
fail2ban_dbmaxmatches: 10
```

### `fail2ban_dbpurgeage`

Type: `raw`. Required: `false`.

Retention period for stored bans, in seconds or Fail2ban duration notation.

Default:

```yaml
fail2ban_dbpurgeage: 86400
```

### `fail2ban_loglevel`

Type: `str`. Required: `false`.

Daemon logging level.

Default:

```yaml
fail2ban_loglevel: INFO
```

### `fail2ban_logtarget`

Type: `str`. Required: `false`.

Log output destination, such as a file path, SYSLOG, or STDERR.

Default:

```yaml
fail2ban_logtarget: /var/log/fail2ban.log
```

### `fail2ban_pidfile`

Type: `str`. Required: `false`.

Daemon process ID file path.

Default:

```yaml
fail2ban_pidfile: /var/run/fail2ban/fail2ban.pid
```

### `fail2ban_socket`

Type: `str`. Required: `false`.

Unix socket path used by fail2ban-client.

Default:

```yaml
fail2ban_socket: /var/run/fail2ban/fail2ban.sock
```

### `fail2ban_stacksize`

Type: `int`. Required: `false`.

Thread stack size in KiB; zero uses the platform default.

Default:

```yaml
fail2ban_stacksize: 0
```

### `fail2ban_syslogsocket`

Type: `str`. Required: `false`.

Syslog socket path; auto selects the platform socket when logging to SYSLOG.

Default:

```yaml
fail2ban_syslogsocket: auto
```

### `fail2ban_jail_local`

Type: `list`. Required: `false`.

INI options in jail.local; removing an option needs no value.

Default:

```yaml
fail2ban_jail_local:
  - section: DEFAULT
    option: bantime
    value: 3600
  - section: DEFAULT
    option: findtime
    value: 600
  - section: DEFAULT
    option: maxretry
    value: 3
  - section: DEFAULT
    option: ignoreip
    value: 127.0.0.1/8 ::1 {{ ansible_facts.default_ipv4.network }}/{{ ansible_facts.default_ipv4.netmask
      }}
  - section: DEFAULT
    option: destemail
    value: root@localhost
  - section: DEFAULT
    option: sender
    value: root@{{ ansible_facts.fqdn }}
  - section: DEFAULT
    option: mta
    value: sendmail
  - section: DEFAULT
    option: action
    value: '%(action_)s'
```

### `fail2ban_actions`

Type: `list`. Required: `false`.

Named action.d/*.local files and their INI options.
Each entry requires name and options; use an empty options list for no changes.

Default:

```yaml
fail2ban_actions: []
```

### `fail2ban_filters`

Type: `list`. Required: `false`.

Named filter.d/*.local files and their INI options.
Each entry requires name and options; use an empty options list for no changes.

Default:

```yaml
fail2ban_filters: []
```

### `fail2ban_jails`

Type: `list`. Required: `false`.

Named jail.d/*.local files and their INI options.
Each entry requires name and options; use an empty options list for no changes.

Default:

```yaml
fail2ban_jails: []
```

## Managed Files

- `/etc/fail2ban/fail2ban.local`
- `/etc/fail2ban/jail.local`
- `/etc/fail2ban/action.d/<name>.local`
- `/etc/fail2ban/filter.d/<name>.local`
- `/etc/fail2ban/jail.d/<name>.local`

## Check Mode

Native modules support check mode on hosts with fail2ban already installed.

- A first installation in check mode cannot create the package-provided
  directories or service.

## Service Behavior

Each run enables and starts fail2ban. Configuration changes notify a restart at
the normal Ansible handler boundary. Repeated runs with unchanged inputs and
service state are idempotent.

### Handlers

- Restart fail2ban after changes to daemon settings, jail options, filters, or
  actions.

## Security Notes

- Configuration files are owned by root with mode 0640 and use module-provided
  backups.
- The default ignoreip setting excludes loopback and the subnet of the default
  IPv4 route.

## Operational Notes

- Entries in fail2ban_actions, fail2ban_filters, and fail2ban_jails require name
  and options.
- Names must be nonempty and contain no path separators or whitespace.
- Every INI option requires section and option; state defaults to present, which
  also requires value.
- Set state to absent to remove an option without supplying value; omitting an
  entry leaves existing settings intact.
- Keep option entries unique per file and section to ensure an idempotent
  desired state.
- No single-file native validator is available for template.validate:
  fail2ban-client -t -c expects a complete configuration directory.
  Configuration validation before service activation is not implemented by this
  role.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Configure an SSH jail

Enable SSH protection using the systemd journal and the packaged sshd
filter and default ban action. The journal backend and action dependencies
must be installed.

```yaml
---
- name: Configure SSH protection
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.fail2ban
      fail2ban_jails:
        - name: sshd
          options:
            - { section: sshd, option: enabled, value: true }
            - { section: sshd, option: backend, value: systemd }
            - { section: sshd, option: maxretry, value: 5 }
```

### Remove a jail override

Remove an explicit bantime option so the jail inherits its default again.

```yaml
---
- name: Restore the inherited ban duration
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.fail2ban
      fail2ban_jails:
        - name: sshd
          options:
            - { section: sshd, option: bantime, state: absent }
```

## References

- [Fail2ban with FirewallD](https://fedoraproject.org/wiki/Fail2ban_with_FirewallD)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2024 Jonas Mauer.
