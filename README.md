# restprof

restprof is a small shell script wrapper for backup utility `restic`. It has
the following features:

- Repository configuration can be placed in sub-directories under
  `/etc/restprof` each of which is a profile.
- The ability to work with read-only file systems:
  - The containing read-only file system is optionally remounted read-write for
    write operations.
  - Read operations can optionally add `--no-lock` when the repository mount is
    read-only.

## Usage

The usage is the same as restic except for the fact that the first argument is
either a profile name, or `-a` indicating that all profiles are to be applied,
and which are applied in alphabetical order.

```
restprof profile command [options]
restprof -a <command> [options]
```

## Requirements

- `restic` usually from the `restic` package.
- `findmnt` usually from the `util-linux` package.

## Install

- Copy `bin/restprof` to a location in your `PATH`.
- Copy `etc` files and directories to the corresponding `/etc` location.

Example installation:
```
cp -a bin/restprof /usr/local/bin
cp -au etc/restprof /etc
cp -a etc/cron.daily/* /etc/cron.daily
cp -a share/* /usr/share/bash-completion/completions
```

## Configuration

The wrapper locates configuration based on the script name and profile:

- Copy `/etc/restprof/template` to the intended profile name in the same
  location. For example, if the profile is to be named `local`:
```
cp -a /etc/restprof/template /etc/restprof/local
```
- Edit files in the newly created profile directory.

Each configuration directory should include:

- `env` - shell snippet that sets required environment variables.
- `files` - paths to back up (one per line, optional).
- `excludes` - exclude patterns (one per line, optional).
- `password` - password file (referenced by `env`). Make sure this is not world
  readable, or writable. Make sure it contains a strong password, and not the
  example default value.

Environment variables:

The wrapper sources `env` before running `restic`.

Required `RESTIC_` variables (example values):

- `RESTIC_REPOSITORY=/mnt/backups/restic`
- `RESTIC_PASSWORD_FILE="$config/password"`

Wrapper-specific variables:

- `RESTPROF_REMOUNT_RW=t` to allow remounting a read-only
  file system read-write before backup for write commands (such as `backup`).
  The wrapper only attempts this for repository paths that start with `/`, and
  refuses to remount `/`.
- `RESTPROF_REMOUNT_NO_LOCK=t` to allow adding `--no-lock` automatically for
  read-only commands when the repository mount is read-only. Set it to `f` to
  disable this behavior.

## Examples

- `restprof local snapshots` lists snapshots based on the configuration in
  `/etc/restprof/local`. This works for read-only file systems.
- `restprof remote backup` performs a backup based on the configuration in
  `/etc/restprof/remote`. This uses the files and excludes as indicated by files
  with those names in the profile, so they don't need to be specified.
- `restprof -a snapshots` lists snapshots for all repositories.
- You can run restprof from cron. For example, `etc/cron.daily/restprof-backup`
  shows a daily backup and retention policy.
