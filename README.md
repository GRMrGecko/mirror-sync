# mirror-sync
A tool to mirror repostories for Linux and other similar tools. This tool is designed to help follow upstream mirror instructions, and implement the features they expect from a downstream official mirror. It also includes features to help keep you in the loop in case of situations that need manual intervention.

## Configuration
It is suggested that you mirror using a sub user account, this tool prevents execution as root to protect you. Once you have an user account dedicated to mirror activities, you can make the log directory, configure logrotate, and add a configuration file to define configurations.

### Making log directory
```bash
mkdir -p /var/log/mirror-sync/
chown mirror: /var/log/mirror-sync/
```

### Configuration for logrotate
```
/var/log/mirror-sync/*.log {
    rotate 7
    create 644 mirror mirror
    daily
    missingok
    notifempty
    sharedscripts
    copytruncate
    compress
}
```

### Configuring mirror-sync
The configuration file is in `/etc/mirror-sync.conf` and is formatted in bash.

## Main configurations
### MODULES
The available modules separated by space. Each module is a separate repostory to sync, and this list allows the script to know how to find their configs.

### TRACEHOST
The hostname to show in trace project files, it defaults to the FQDN hostname of the server.

### mirror_hostname
The hostname of this mirror server, it defaults to the FQDN hostname of the server. If you have a public domain for your mirror, you may wish to adjust this configurtion to that.

### PIDPATH
If you wish to override where pid files are stored to prevent duplicate module syncs, the default is `/tmp` and the directory must have write access for the mirror user.

### LOGPATH
If you wish to override where logs are stored, the default is `/var/log/mirror-sync` and the directory must have write access for the mirror user.

### sync_timeout
Timeout before a sync is cancelled, defaults to `timeout 1d` which should work for most mirrors.

### max_errors
How many errors before an email is sent regarding the issue. This allows you to ignore anomolies.

### upstream_max_age
If the upstream last modified date is older than the defined number of seconds, the upstream check will skip syncing. Default is 5 hours.

### upstream_timestamp_min
If an upstream check is configured, this defines the minimum age in seconds of the last successful sync before the next sync will skip the upstream check. Default is 24 hours.

### QFM_PATH
Path to where quick-fedora-mirror is located and configurations are saved. If you already have QFM installed, but want configurations stored separately. You can use the `QFM_BIN` configuration to set the QFM binary path.

### QFM_BIN
The binary path for quick-fedora-mirror. If you override `QFM_PATH`, you will likely also have to override this path. Default:
```bash
QFM_BIN="$QFM_PATH/quick-fedora-mirror"
```

### JIGDO_FILE_BIN
If you installed jigdo outside of the home directory, you need to manually configure the `jigdo-file` binary path here.

### JIGDO_MIRROR_BIN
If you installed jigdo outside of the home directory, you need to manually configure the `jigdo-mirror` binary path here.

### jigdoConf
If you use jigdo to build ISO images, this is the base configuration file name. The jigdo hook saves configurations in `${jigdoConf:?}.${arch}.${s}` format.

### MAILTO
The email address of which to mail errors to.

### INFO_MAINTAINER
The maintainer of this repository, should be defined in `name <email>` format.

### INFO_SPONSOR
If this repo is sponsored, you may define the sponsors here.

### INFO_COUNTRY
The country of which this server resides.

### INFO_LOCATION
The region of which this server resides (state/providence).

### INFO_THROUGHPUT
How fast are the pipes to your repository.

### INFO_TRIGGER
How did the sync occur, cron job or manually via ssh? This is auto detected and you do not need to define this configuration.

### dusum_human_readable_total_file
Path to save a grand total of each disk usage sum in human readable form.

### dusum_kbytes_total_file
Path to save a grand total of each disk usage sum in killo bytes.

## Module specific configurations
Each module is configured via configurations prefixed by the module name. The one configuration used by all modules is the `_sync_method` configuration which defines what sync method to use. Each sync method has different configurations available. The default sync method is rsync.

Each repo has at bare minimum the following configurations:

- sync_method - rsync, git, aws, s3cmd, ftp, wget, or qfm.
- repo - The destination directory of the repository.
- timestamp - Path to a file to store the last successful sync unix time stamp. Can be used by a monitoring system to confirm each repo is syncing successfully.
- dusum - Path to a file to store disk usage summary results of the repository directory.

### git
Synchronizes a git repository via git pull. To use this method, you need to have the git package installed.

#### options
Extra options appended to `get pull`.

#### Example
```bash
example_sync_method="git"
example_repo="/home/mirror/http/example"
example_timestamp="/home/mirror/timestamp/example"
```

### aws
Synchronize with an s3 bucket using aws cli. To use this, you need the aws cli package installed.

#### aws_bucket
The bucket URL to sync with.

#### aws_access_key
The access key for the s3 bucket.

### aws_secret_key
The secret for the s3 bucket.

#### aws_endpoint_url
If you are using a third party S3 compatible service, you can enter their endpoint URL here.

#### options
Extra options to append to `aws s3 sync`.

#### Example
```bash
example_sync_method="aws"
example_repo="/home/mirror/http/example"
example_timestamp="/home/mirror/timestamp/example"
example_aws_bucket="s3://bucket/directory"
example_aws_access_key="RANDOM_KEY_FROM_PROVIDER"
example_aws_secret_key="RANDOM_SECRET_FROM_PROVIDER"
```

### s3cmd
Synchronize with an s3 bucket using s3cmd. To use this, you need the s3cmd package installed.

#### aws_bucket
The bucket URL to sync with.

#### aws_access_key
The access key for the s3 bucket.

### aws_secret_key
The secret for the s3 bucket.

#### options
Extra options to append to `s5cmd`.

#### Example
```bash
example_sync_method="s3cmd"
example_repo="/home/mirror/http/example"
example_timestamp="/home/mirror/timestamp/example"
example_aws_bucket="s3://bucket/directory"
example_aws_access_key="RANDOM_KEY_FROM_PROVIDER"
example_aws_secret_key="RANDOM_SECRET_FROM_PROVIDER"
```

Example of using third party bucket:
```bash
example_sync_method="s3cmd"
example_repo="/home/mirror/http/example"
example_timestamp="/home/mirror/timestamp/example"
example_aws_bucket="s3://bucket/directory"
example_aws_access_key="RANDOM_KEY_FROM_PROVIDER"
example_aws_secret_key="RANDOM_SECRET_FROM_PROVIDER"
example_options="--host='objects.example.com' --host-bucket='%(bucket).objects.example.com'"
```

### s5cmd
Synchronize with an s3 bucket using s5cmd. The s5cmd will auto install if not existing.

#### aws_bucket
The bucket URL to sync with. You must end the bucket url with `*` for s5cmd to work.

#### aws_access_key
The access key for the s3 bucket.

### aws_secret_key
The secret for the s3 bucket.

#### aws_endpoint_url
If you are using a third party S3 compatible service, you can enter their endpoint URL here.

#### options
Extra options to append to `s5cmd`.

#### sync_options
Extra options to append to the `sync` command of s5cmd.

#### Example
```bash
example_sync_method="s5cmd"
example_repo="/home/mirror/http/example"
example_timestamp="/home/mirror/timestamp/example"
example_aws_bucket="s3://bucket/directory/*"
example_aws_access_key="RANDOM_KEY_FROM_PROVIDER"
example_aws_secret_key="RANDOM_SECRET_FROM_PROVIDER"
```

### ftp
Synchronize both http and ftp sources to a repo. This sync method requires the lftp package to be installed.

#### source
The source url to mirror from.

#### options
Extra options to append to the mirror command of lftp.

#### Example
```bash
example_sync_method="ftp"
example_repo="/home/mirror/http/example/"
example_timestamp="/home/mirror/timestamp/example"
example_source="https://repos.example.com/rhel/7/x86_64/stable"
```

### wget
Synchronizes using wget to a repository. To use this, you need the wget package installed.

#### source
The source url to mirror from.

#### options
The options passed to wget. Defaults to `--mirror --no-host-directories --no-parent`.

#### Example
```bash
example_sync_method="wget"
example_repo="/home/mirror/http/example/"
example_timestamp="/home/mirror/timestamp/example"
example_source="https://repos.example.com/rhel/7/x86_64/stable"
example_options="--mirror --no-host-directories --no-parent --cut-dirs=4"
```

### rsync
By far, the most common mirror method is to use rsync. It, while not perfect, is more efficent than using wget or ftp mirroring. You will need the rsync package installed for this to function. There is an extra CLI argument available for this sync method, `--force` which allows you to by-pass upstream checks and synchronize immediately.

#### pre_hook
A hook to run prior to the first stage sync.

#### source
The rsync server or ssh server URL.

#### options
Synchronization options for the first rsync stage.

#### options_stage2
If your repo needs a 2 stage rsync, define some options here. The most basic option you can use, if you want to force stage 2 to occur, would be `--exclude '.~tmp~'`.

#### pre_stage2_hook
A hook to run prior to the second stage sync.

#### upstream_check
An http URL to check the last modified date as a reference for if the upstream mirror was possibly modified recently. This option is mainly here to lower the impact on upstream mirrors so that mirrorning happens less often. See `upstream_timestamp_min` and `upstream_max_age` for global configuration options of this check.

### time_file_check
Name of a time file to check if the upstream has updated before syncing all files to reduce load on upstream mirrors.

#### report_mirror
If you have Fedora report mirror installed, and need to report back to Fedora about the status of your repository, you can provide this option a configuration path for the `report_mirror` utility to run the report after a successful sync.

#### rsync_password
If you have an rsync password and need to authenticate with an rsync server, this is where you define the password.

#### post_hook
Any hooks to call after a successful sync, define here. If you are using jigdo, the hook is `jigdo_hook`.

#### jigdo_pkg_repo
If you are using jigdo to build ISO images, you need to define the path to the repo of packages.

#### arch_configurations
Information for trace files on what architectures are synchronized to this mirror.

#### type
For the trace file saving, this defines what type of repo is being synced. Options are deb, rpm, iso, or source.

#### Example
Example for RPM based mirror:
```bash
example_repo="/home/mirror/http/example/"
example_timestamp="/home/mirror/timestamp/example"
example_source="rsync://rsync.example.org/module/"
example_options="--exclude '.~tmp~' --exclude 'repodata/*'"
example_options_stage2="--exclude '.~tmp~'"
example_type="rpm"
```

Example for DEB based mirror:
```bash
example_repo="/home/mirror/http/example/"
example_timestamp="/home/mirror/timestamp/example"
example_source="rsync://rsync.example.org/module/"
example_options="--exclude '.~tmp~' --include=*.diff/ --exclude=*.diff/Index --exclude=Packages* --exclude=Sources* --exclude=Release* --exclude=InRelease --include=i18n/by-hash --exclude=i18n/* --exclude=ls-lR*"
example_options_stage2="--exclude '.~tmp~'"
example_type="deb"
```

Example with jigdo:
```bash
example_repo="/home/mirror/http/example/"
example_timestamp="/home/mirror/timestamp/example"
example_source="rsync://rsync.example.org/module/"
example_options="--exclude '.~tmp~' --exclude '*.iso'"
example_pre_stage2_hook="jigdo_hook"
example_jigdo_pkg_repo="/home/mirror/http/debian/"
example_options_stage2="--exclude '.~tmp~'"
example_type="iso"
```

### qfm
Quick Fedora Mirror is a tool to help Fedora mirrors distribute changes faster and save on resources when trying to discover what needs to be synced. To use this method, you must have both the rsync and zsh package installed. This tool automatically downloads QFM if you do not already have it installed.

This tool requires that the upstream mirror has an module with sub modules designed for use with quick-fedora-mirror. You can use this tool with non-fedora mirrors, however they must follow the fedora module configurations. For fedora mirrors, you can utilize [tier 1 mirrors](https://fedoraproject.org/wiki/Infrastructure/Mirroring/Tiering#Tier_1_mirrors).

You can list modules available on an rsync server with:
```bash
rsync --list-only rsync://SERVER
```

And to check a module out, you can list the files with:
```bash
rsync --list-only rsync://SERVER/MODULE
```

#### repo
For the repo config, QFM requires the directory to be `$DOCROOT` which it'll then copy modules into. This is different from all other sync methods.

#### pre_hook
A hook to run prior to running QFM.

#### source
The source rsync server, without any modules appended.

#### master_module
The main rsync module under which the fedora sub module directories exist. Defaults to `fedora-buffet`.

#### module_mapping
If you are using this with a non-fedora mirror, you can define your own custom sub module mapping.

#### mirror_manager_mapping
The names for custom module mapping.

#### modules
The sub modules to sync. It is recommended that you only do one sub module, the modules available by default are fedora-alt, fedora-archive, fedora-enchilada, fedora-epel, and fedora-secondary.

#### options
Extra options to pass to quick-fedora-mirror.

#### filterexp
If you wish to filter out particular directories/files, define regular expression here.

#### rsync_options
Extra options to pass to rsync during sync.

#### report_mirror
If you have Fedora report mirror installed, and need to report back to Fedora about the status of your repository, you can provide this option a configuration path for the `report_mirror` utility to run the report after a successful sync.

#### rsync_password
If you have an rsync password and need to authenticate with an rsync server, this is where you define the password.

#### post_hook
Any hooks to call after a successful sync, define here.

#### arch_configurations
Information for trace files on what architectures are synchronized to this mirror.

#### type
For the trace file saving, this defines what type of repo is being synced. Options are deb, rpm, iso, or source.

#### Example
```bash
example_sync_method=qfm
example_repo='/home/mirror/http/'
example_timestamp='/home/mirror/timestamp/example'
example_source='rsync://mirrors.example.com'
example_modules=fedora-enchilada
example_report_mirror='/home/mirror/report_mirror.conf'
example_type=rpm
```

## CLI Options
There are not that many cli options available, usage is as follows:
```
[--help|--update-support-utilities] {module} [--force]
```

## Requirements list

- bash
- zsh
- sendmail
- git
- awscli
- s3cmd
- lftp
- wget
- curl
- rsync
- jigdo - this tool auto installs.
- quick-fedora-mirror - this tool auto installs.

### Install on RPM based servers
```bash
yum install bash zsh sendmail git awscli s3cmd lftp wget curl rsync
```

### Install on DEB based servers
```bash
apt install bash zsh sendmail git awscli s3cmd lftp wget curl rsync
```

### Install on Arch 
```bash
yay -S bash zsh sendmail git aws-cli-git s3cmd lftp wget curl rsync
```
