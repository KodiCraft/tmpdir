# tmpdir

`tmpdir` is a command line utility for creating temporary directories on Linux systems.

It is very quick to use and is an alternative to `mktemp -d` which attempts to be more user friendly.

## Quick start

Simply use this command to create a new temporary directory in the current working directory:

```bash
$ tmpdir tmp # root privileges are required
```

To clear up all temporary directories once they are no longer needed, use this command:

```bash
$ tmpdir -c # root privileges are required
```

`tmpdir -c` behaves in a way that makes it safe to use automatically in something like a cron job or on system startup. It will only remove directories that were created by `tmpdir` and protect any folder that still contains files after being unmounted.

## Installation

`tmpdir` is a bash script and can be installed by simply copying it to a directory in your `$PATH`:

```bash
$ sudo cp tmpdir /usr/local/bin
```

All of its dependencies should be standard GNU utilities that are already installed on most Linux systems. Just to make sure you have everything you need, here is a list of the binaries that `tmpdir` uses:
- `bash`
- `df`
- `tail`
- `awk`

## Usage

```
tmpdir [options] <directory>
============================
<directory> - Path to the temporary directory to create
========Options=============
-v / --verbose - Be extra verbose
-h / --help - Display this help message
-c / --clear - Clear all temporary directories, create nothing
-l / --list - List all temporary directories, create nothing
-f / --file FILE - Path to the persistent data file, defaults to /etc/tmpdir.conf
-t / --type TYPE - Type to mount on the directly (tmpfs, ramfs, etc), defaults to tmpfs
-s / --size SIZE - Size of the directory to create, defaults to 1G
-m / --mode MODE - Mode to mount the directory with, defaults to 1777
============================
Please note that this tool does not perform option unpacking, for instance:
-vc will not work, you must use -v -c
```

## Persistent data

`tmpdir` stores a list of all temporary directories that it has created in a file called `/etc/tmpdir.conf`. This file is used to determine which directories to clear when `tmpdir -c` is run. This file can be changed by using the `-f` option.

## Behavior

Directories are mounted as `tmpfs` by default and will therefore be automatically cleared when the system is rebooted or when the directory is unmounted. `tmpdir` does not officially support other types of mount but the `-t` option can be used to change the mount type if you wish to experiment.

## License

`tmpdir` is licensed under the MIT license. See the [LICENSE](LICENSE) file for more information.