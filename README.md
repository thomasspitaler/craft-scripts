# craft-scripts

Scripts for backing up and restoring the database and assets of [Craft cms](https://craftcms.com/) projects. There is also provided a script for setting the proper file system permissions.

Currently only the local file system and [Google Cloud Storage](https://cloud.google.com/storage) are supported for storing backups, but the scripts can easily be extended to use other storage types.

These scripts are not concerned with authentication. It is assumed that the machines that these scripts are run on are already authenticated and authorized to access the storage used to store backups.

## Installation

Clone or copy the content of this repository into a directory within you project, for example named `bin`.

Copy the provided configuration example `cp .env.example .env` and set the variables in `.env` according to your environment, see next section for details.

## Configuration

The scripts are configured via en `.env` file, as follows:

```bash
PATH_TO_SITE="/Users/thomas/code/your-app" # absolute path to site, without trailing slash
ASSETS_DIR="web/content" # directory of assets to be backed up and restored

FS_USER="thomas" # file system user for chown/chmod (set-perms script)
FS_GROUP="staff" # file system group for chown/chmod (set-perms script)

# database credentials
DB_DRIVER="pgsql" #  currently only 'pgsql' (PostgreSQL) and 'mysql' (MySQL) are supported
DB_SERVER="localhost"
DB_USER="dev"
DB_PASSWORD="dev"
DB_DATABASE="your_app"
DB_PORT="5432"

# storage for storing backups
STORAGE_TYPE="gs" # 'gs' (google cloud storage) or 'fs' (local file system)
STORAGE_ROOT="gs://your-backups"
```

## Running the Scripts

The following scripts are provided:

- `backup-database`: backs up the database

- `backup-assets`: backs up craft assets (`web/cpresources`and the directory configured via `ASSETS_DIR`)

- `backup`: runs both `backup-database` and `backup-assets`

- `restore-database`: restores the database

    The script wil look for the most current backup and prompt the user whether it is ok to restore that backup.

- `restore-assets`: restores craft assets (`web/cpresources`and the directory configured via `ASSETS_DIR`)

    The script wil look for the most current backup and prompt the user whether it is ok to restore that backup.

- `restore`: runs both `restore-database` and `restore-assets`

- `set-perms`: sets file system permissions (user and group configured via `FS_USER` and `FS_GROUP`)

## Extending the Scripts

The scripts can easily be extended by providing your own scripts for additional storage and database types.

If you want to add a database type `xy`, in addition to setting `DB_DRIVER=xy`, scripts `private/database-dump-xy` and `private/database-restore-xy` need to be implemented.

Additional storage types, say `xy`, are added by setting `STORAGE_TYPE=xy` as well as implementing scripts `private/storage-list-xy` and `private/storage-copy-xy`.
