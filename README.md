ServerBackup
============

[![Test](https://github.com/tilfin/serverbackup/actions/workflows/test.yml/badge.svg)](https://github.com/tilfin/serverbackup/actions/workflows/test.yml)

Server backup to cloud

It can rotate the bucket which is backup destination of each day.


Prerequisites
-------------

Any sync tool

For example,

* s3sync (easy to use https://github.com/tilfin/s3sync forked from aproxacs/s3sync)
* Google Cloud CLI (`gcloud storage`; https://cloud.google.com/sdk/docs/install)


Setup
-----

```
$ sudo -i
# mkdir -p /backup/log
# mkdir /backup/tmp
# cd /backup
# git clone --depth 1  https://github.com/tilfin/serverbackup.git
# cd serverbackup
# cp backup.conf.sample backup.conf
```

### Quick Settings for Google Cloud Storage

#### Install and authenticate the Google Cloud CLI

Install the [Google Cloud CLI](https://cloud.google.com/sdk/docs/install) on the backup host and check that `gcloud storage` is available. Run authentication as the same OS user that runs the cron job (the example below uses `root`). Give that identity `roles/storage.objectUser` on each destination bucket.

* On a Google Compute Engine VM, attach a service account to the VM and use the `cloud-platform` access scope. The CLI uses the attached service account automatically.
* On a host outside Google Cloud, use [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation) and run `gcloud auth login --cred-file=/path/to/credential-config.json` as the backup user. Keep the credential configuration available when the job runs.

The old `boto.cfg` and P12 key are not used by `gcloud storage`. After verifying the new authentication, remove them from `/backup/serverbackup`: the backup script archives every file in that directory.

#### backup.conf

Edit `Buckets` and `Sync` in `/backup/serverbackup/backup.conf`:

```
Buckets=backup-bucket
Sync=/backup/serverbackup/sync.gcloud
```

Existing `Sync=/backup/serverbackup/sync.gsutil` settings still work; that script now calls `sync.gcloud`. Syncing retains the existing remote objects because it does not use `--delete-unmatched-destination-objects`. `backup -n` previews the Cloud Storage sync with `--dry-run`.

#### Backup commands

bkcmd.d behaves like _rcX.d_. Kick each script with prefix 'S' in this directory.
${BKUP_DIR} is a private directory created for each run under `BackupTmpDir`.
${BKUP_PREFIX} adds the day number to that directory (for example, _/backup/tmp/backup.A1B2C3D4/3-_). The run directory is removed when backup exits. Files already present directly under `BackupTmpDir` are left alone.
`BackupTmpDir` must be an existing directory owned by the backup user without group or other write permission. Shared directories such as `/tmp` are not supported because the lock file is stored there.


#### Crontab

Kick backup at 3:00am

```
0 3 *   *   *     /backup/serverbackup/backup
```

You can mail a backup result if you set MAILTO in crontab.
