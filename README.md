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

The above method is that the token refresh does not work.
It is permanently set up in the following way.

1. Create a service account at your Google cloud console.
2. Select to furnish a new private key whose type is _P12_.
3. Put created .p12 file at `/backup/serverbackup/<secret key file.p12>`
4. Write `/backup/serverbackup/boto.cfg` with the content of the following

```
[Credentials]
gs_service_client_id = <service account email address>
gs_service_key_file = /backup/serverbackup/<secret key file.p12>
gs_service_key_file_password = <pass phrase for key file>

[Boto]
https_validate_certificates = True

[GSUtil]
content_language = en
default_api_version = 2
default_project_id = <Google Developer Project ID>
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
