ServerBackup
============

[![Build Status](https://travis-ci.org/tilfin/serverbackup.svg)](https://travis-ci.org/tilfin/serverbackup)

Server backup to cloud

It can rotate the bucket which is backup destination of each day.


Prerequisites
-------------

Any sync tool

For example,

* s3sync (easy to use https://github.com/tilfin/s3sync forked from aproxacs/s3sync)
* gsutil (https://cloud.google.com/storage/docs/gsutil)


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

#### Install gsutil

```
# apt-get install python-dev python-crypto
# cd /backup
# wget https://storage.googleapis.com/pub/gsutil.tar.gz
# tar zxf gsutil.tar.gz
```

#### Setup gsutil authentication

```
# /backup/gsutil/gsutil config -o /backup/serverbackup/boto.cfg
```

#### backup.conf

Edit following entries.

* Buckets
* Sync

```
Buckets=backup-bucket
Sync=/backup/serverbackup/sync.gsutil
```

#### Backup commands

bkcmd.d behaves like _rcX.d_. Kick each script with prefix 'S' in this directory.
${BKUP_PREFIX} is combined backup directory path and file prefix which is the day number (ex _/backup/tmp/3-_).


#### Crontab

Kick backup at 3:00am

```
0 3 *   *   *     /backup/serverbackup/backup
```

You can mail a backup result if you set MAILTO in crontab.
