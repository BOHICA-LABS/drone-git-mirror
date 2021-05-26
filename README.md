# git-mirror

## About

The plugin is used for mirroring Git repositories to external ones and it has 2 modes:

1. full mirroring (default)
1. partial mirroring

Details on both of them are described below.

## Usage example

### Full mirroring

By default, this plugin will perform full mirroring. Please note, that the clone step should be **disabled**.

``` yaml
trigger:
  event: [push, tag]

clone:
  disable: true

steps:
  - name: mirror
    image: bohicalabs/drone-git-mirror
    settings:
      target_repo: git@github.com:bohicalabs/some-repo.git
      ssh_key:
        from_secret: git-mirror-private-key
```

This mode works similar to `--mirror` option for `git-push`. Technical details you can find [here](entrypoint.sh#L83-L88).

### Partial mirroring

If the file named `mirror_ignore_list` exists, then the plugin will perform partial mirroring.

``` yaml
steps:
  - name: mirror
    image: bohicalabs/drone-git-mirror
    settings:
      target_repo: git@github.com:bohicalabs/some-repo.git
      ssh_key:
        from_secret: git-mirror-private-key
```

The content of `mirror_ignore_list` file should be compatible with `--exclude-from` option from `rsync` flag (GNU tar way).

## Params

| Param | Default | Description |
| -------- | -------- | -------- |
| `target_repo` | - | SSH clone URL of target repo |
| `ssh_key` | - | Private key with R/W permissions to use for interacting with `target_repo`. |
| `git_email` | `drone@bohicalabs` | E-mail that will be used while pushing changes |
| `git_name` | `Drone CI` | Name that will be used while pushing changes |
| `ignore_errors` | `false` | If set to `true` the plugin will try to ignore all occurring errors to prevent build failing because of itself. |
| `mirror_ignore_list` | `.mirror_ignore` | If this file is found in repo root - it contents will be used as exclusion list for mirroring |

## FAQ

### How to exclude unwanted files from sync

You need to create a file, named `.mirror_ignore` (if not overwritten with `mirror_ignore_list` variable) in the root directory of your repository. Describe wanted and unwanted files and directories inside as if you are preparing a file for using with `--exclude-from` option from `rsync` (GNU tar way).

For example:

``` plaintext
.drone.yml
.mirror_ignore
file1.txt
dir1/*
dir2
```

### How to delete excluded files or branches from remote repo

Files, excluded with `mirror_ignore_list` will not be automatically removed from the remote repository. Neither do branches. No plans of implementing such functionality. Hence, you should do it manually.

## TODO

- hide the private key in debug output
- add the way to override the target branch in the remote repository (master pinned for now)
- deal with tags not being mirrored in "partial" scenario
