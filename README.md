# rclone-mount

Mount cloud storage using [rclone](https://rclone.org/),
[unionfs-fuse](https://github.com/rpodgorny/unionfs-fuse), and
[Docker](https://www.docker.com/).

`rclone`'s FUSE support is **EXPERIMENTAL** (see
[here](https://rclone.org/commands/rclone_mount/)), so the `rclone` remote is
mounted as read-only.  `unionfs-fuse` adds a writable layer, using a local
directory to cache changes.  Scripts run on a schedule to persist changes to
the `rclone` remote.

```
docker run -d --name rclone-mount \
    --cap-add SYS_ADMIN \
    --device /dev/fuse \
    -e RCLONE_REMOTE="" \
    -v /local/path/to/rclone.conf:/root/.rclone.conf \
    -v /local/mount/target:/mnt/unionfs:shared \
    jdavis92/rclone-mount
```

### Required:

Access to FUSE:
```
--cap-add SYS_ADMIN \
--device /dev/fuse
```

Name of `rclone` remote to mount (must be defined in rclone.conf):
```
-e RCLONE_REMOTE=""
```

`rclone` config:
```
-v /local/path/to/rclone.conf:/root/.rclone.conf
```

### Optional:

Mount the `unionfs-fuse` directory:
```
-v /local/mount/target:/mnt/unionfs:shared
```

Mount the local file cache.  Prevents data loss if the docker container goes
down before local changes are persisted to the `rclone` remote:
```
-v rclone_local:/tmp/local
```

Change the CRON schedule for persisting changes to the `rclone` remote (see
[here](https://en.wikipedia.org/wiki/Cron#CRON_expression) for help with CRON
expressions):
```
-e SCHEDULE="0 9 * * *"
```

Change UID and GID of mounted files:
```
-e MOUNT_UID="1000" \
-e MOUNT_GID="1000"
```

Allow docker containers to use FUSE on systems with AppArmor enabled:
```
--security-opt apparmor:unconfined
```
