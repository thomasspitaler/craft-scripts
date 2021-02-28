# craft-scripts

Scripts for backing up and restoring the database and assets of [Craft cms](https://craftcms.com/) projects.

There are also provided scripts for setting the proper file system permissions and to clear the Craft cms cache.

Currently only the local file system and [Google Cloud Storage](https://cloud.google.com/storage) are supported for storing backups, but the scripts can easily be extended to use other storage types.

Some storage types like for example Google Cloud Storage need authentication, and there is also a script to handle that.

## Installation

Clone or copy the content of this repository into a directory within you project, for example named `bin`.

Copy the provided configuration example `cp .env.example .env` and set the variables in `.env` according to your environment, see next section for details.

## Configuration

The scripts are configured via en `.env` file, as follows:

```bash
PROJECT_NAME="my-project" # will be used to name the generated archives
PROJECT_ROOT="/Users/thomas/code/my-project/cms" # absolute path to site, without trailing slash

FS_USER="thomas" # file system user for chown/chmod (set-perms script)
FS_GROUP="staff" # file system group for chown/chmod (set-perms script)

# archive of assets will be built from these directories
# permissions will be set on these as well
ASSET_DIRS=(
    "web/cpresources"
    "web/content"
)

# database credentials, similar to the ones found in the .env file used by Craft cms
DB_DRIVER="pgsql" #  currently only 'pgsql' (PostgreSQL) and 'mysql' (MySQL) are supported
DB_SERVER="localhost"
DB_PORT=5432
DB_DATABASE="your_database"
DB_USER="your-user"
DB_PASSWORD=""

# storage for storing backups
STORAGE_TYPE="gs"
STORAGE_ROOT="gs://backups.my-project.com" # 'gs' (google cloud storage) or 'fs' (local file system)

# config specific to storage type "gs"
# google storage access key (json)
STORAGE_GS_ACCESS_KEY='{
  "type": "service_account",
  "project_id": "your-project-id",
  "private_key_id": "85fe26393cffbc5dff14588db138f249073cb766",
  "private_key": "key data",
  "client_email": "assets-your-project-com@your-project.iam.gserviceaccount.com",
  "client_id": "123455",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/assets-your-project-com%40your-project.iam.gserviceaccount.com"
}
'
```

## Running the Scripts

The following scripts are provided:

- `storage-authenticate`: authentication, needed for some storage types like for example Google Cloud Storage

- `backup-database`: backs up the database

- `backup-assets`: backs up craft assets directories configured via `ASSET_DIRS`)

- `ls-database-backups`: lists the available database backups

- `ls-asset-backups`: lists the available asset backups

- `backup`: runs both `backup-database` and `backup-assets`

- `restore-database [archive]`: restores the database from `archive` (absolute asset paths output by `ls-database-backups`)

    If no archive is provided, the script wil look for the most current backup.

- `restore-assets [archive]`: restores the directory configured via `ASSET_DIRS` from `archive` (absolute asset paths output by `ls-asset-backups`)

    If no archive is provided, the script wil look for the most current backup.

- `restore`: runs both `restore-database` and `restore-assets` without parameters (this will restore from the most current backups)

- `set-perms`: sets file system permissions (user and group configured via `FS_USER` and `FS_GROUP`)

- `clear-cache`: clears the Craft cms cache

## Examples

```bash
$ ./backup-assets
Backing up assets of project my-project.
Creating archive of assets...
Copying archive...
Copying file:///var/folders/67/xfs0v6s96cq7nzvcggzg5bgc0000gn/T/tmp.C8Lxmdid/my-project_20210227T231144.tar.gz [Content-Type=application/x-tar]...
/ [1 files][  9.8 MiB/  9.8 MiB]
Operation completed over 1 objects/9.8 MiB.
Cleaning up

$ ./ls-asset-backups
gs://backups.my-project.com/my-project_20210227T231127.tar.gz
gs://backups.my-project.com/my-project_20210227T231144.tar.gz

$ ./restore-assets gs://backups.my-project.com/my-project_20210227T231144.tar.gz
Restoring assets of project my-project.
Looking for backup gs://backups.my-project.com/my-project_20210227T231144.tar.gz
Found gs://backups.my-project.com/my-project_20210227T231144.tar.gz
Copying assets dump...
Copying gs://backups.my-project.com/my-project_20210227T231144.tar.gz...
\ [1 files][  9.8 MiB/  9.8 MiB]
Operation completed over 1 objects/9.8 MiB.
Restoring assets...
```

## Extending the Scripts

The scripts can easily be extended by providing your own scripts for additional storage and database types.

If you want to add a database type `xy`, in addition to setting `DB_DRIVER=xy`, scripts `private/database-dump-xy` and `private/database-restore-xy` need to be implemented.

Additional storage types, say `xy`, are added by setting `STORAGE_TYPE=xy` as well as implementing scripts `private/storage-list-xy` and `private/storage-copy-xy`.

If the storage type needs authentication, `private/storage-authenticate-xy` needs to be implemented.
