# Sugar Dockerized
This repository will help you deploy a Docker based development full stack for Sugar, meeting all the platform requirements for a different set of platform combinations.

## Stacks available
There are few stacks available, with in itself multiple platform combinations. You can read more about the specific stacks on the links below:
* [Sugar 81](stacks/sugar81/README.md) - For local development to apply to Sugar Cloud only versions
* [Sugar 8](stacks/sugar8/README.md)
* [Sugar 710 or Sugar 711](stacks/sugar710/README.md) - For local development to apply to Sugar Cloud only versions
* [Sugar 79](stacks/sugar79/README.md)
* [Sugar 79 upgraded to a future version](stacks/sugar79upgrade/README.md)

### Types of stacks
There are mainly two types of stack:
* Single Apache web server - Initial development
* An Apache load balancer with a cluster of two Apache web servers behind it - A more real-life environment to verify that everything works correctly

### All stack components
There are multiple stack components as docker containers, that perform different duties. Not all the stack components might be used on a specific stack setup.
* Apache load balancer - Load balances requests between the cluster of Apache PHP web servers, round robin
* Apache PHP web server - Web server(s)
* MySQL database - Database
* Elasticsearch - Sugar search engine
* Redis - Two purposes: Sugar object caching service and PHP Session storage/sharing service
* Cron - Sugar background scheduler processing. Note that this is enabled immediately and it will run `cron.php` as soon as the file is available, and it will attempt to do so every 60 seconds since its last run. This container is used for any other CLI execution required during development
* Permission - Sets Sugar instance permissions correctly and then terminates
* LDAP - LDAP testing server if needed with authentication

## Get the system up and running
* The first step for everything to work smoothly, is to add on your computer's host file /etc/hosts the entry "docker.local" to point to your machine's ip (it might be 127.0.0.1 if running the stack locally or within the VM running Docker)
* Clone the repository with `git clone https://github.com/esimonetti/SugarDockerized.git sugardocker` and enter sugardocker with `cd sugardocker`
* Select the stack combination to run by choosing the correct yml file within the subdirectories inside [stacks](stacks/). See next step for more details and an example.
* Run `docker-compose -f <stack yml filename> up -d` for the selected <stack yml filename>. As an example if we selected `stacks/sugar8/php71.yml`, you would run `docker-compose -f stacks/sugar8/php71.yml up -d`

## Current version support
The main stacks work with [Sugar version 8.0 and all its platform requirements](http://support.sugarcrm.com/Resources/Supported_Platforms/Sugar_8.0.x_Supported_Platforms/). Additional stacks are aligned with the platform requirements of version [7.9](http://support.sugarcrm.com/Resources/Supported_Platforms/Sugar_7.9.x_Supported_Platforms/) and the Sugar Cloud only versions: 7.10/7.11 and 8.1.

## Starting and stopping the desired stack
* Run the stack with `docker-compose -f <stack yml filename> up -d`
* Stop the stack with `docker-compose -f <stack yml filename> down`

## System's details

### Stack components hostnames
* Apache load balancer: sugar-lb
* Apache PHP web server: On single stack: sugar-web1 On cluster stack: sugar-web1 and sugar-web2
* MySQL database: sugar-mysql
* Elasticsearch: sugar-elasticsearch
* Redis - sugar-redis
* Cron - sugar-cron
* Permission - sugar-permissions
* LDAP - sugar-ldap

To verify all components hostnames just run `docker ps` when the stack is up and running.

Please note that on this setup, only the web server or the load balancer (if in single web server or cluster stack) and the database can be accessed externally. Everything else is only allowed within the stack components.

### Core stack components
* Linux
* Apache
* MySQL
* PHP
* Redis
* Elasticsearch

### Sugar Setup details
* Browser url: http://docker.local/sugar/ - Based on the host file entry defined above on the local machine
* MySQL hostname: sugar-mysql
    * MySQL user: root
    * MySQL password: root
* Elasticsearch hostname: sugar-elasticsearch
* Redis hostname: sugar-redis

### Apache additional information
Apache web servers have enabled:
* mod_headers
* mod_expires
* mod_deflate
* mod_rewrite

### PHP additional information
Apache web servers have PHP with enabled:
* Zend OPcache - Configured for Sugar with the assumption the files will be located within the correct path
* xdebug
    * If you use IDE such as PHPStorm, you can setup DBGp Proxy in Preference -> Language & Framework -> PHP -> Debug -> DBGp Proxy. Here is an example setting: <img width="1026" alt="screen shot 2018-04-18 at 10 17 53 pm" src="https://user-images.githubusercontent.com/361254/38972661-d48661f6-4356-11e8-9245-ad598239fe94.png">
* XHProf or Tideways profilers depending on the version

Session storage is completed leveraging the Redis container.

## Elasticsearch additional information
If you notice that your Elasticsearch container is not running (check with `docker ps`), it might be required to tweak your Linux host settings.
To be able to run Elasticsearch version 5 and above, it is needed an [increase of the maximum mapped memory a process can have](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/vm-max-map-count.html). To complete this change permanently run:

`echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf`

Alternatively the limit can be increased runtime with:

`sudo sysctl -w vm.max_map_count=262144`

### Docker images
* `images/elasticsearch/175/` - Elasticsearch 1.7.5
* `images/elasticsearch/54/` - Elasticsearch 5.4
* `images/elasticsearch/56/` - Elasticsearch 5.6
* `images/elasticsearch/62/` - Elasticsearch 6.2
* `images/ldap/` - OpenLDAP
* `images/loadbalancer/` - Apache load balancer
* `images/mysql/57/` - MySQL 5.7
* `images/permissions/` - Permissions fixing container image
* `images/php/56/apache/` - Apache with PHP 5.6
* `images/php/56/cron/` - PHP 5.6 CLI for background jobs
* `images/php/71/apache/` - Apache with PHP 7.1
* `images/php/71/cron/` - PHP 7.1 CLI for background jobs

All images are currently leveraging Debian linux.

### Persistent storage locations
All persistent storage is located within the `./data` directory tree within your local checkout of this git repository.
* The Sugar application files served from the web servers and leveraged by the cronjob server have to be located in `./data/app/sugar/`. Within the web servers and the cronjob server the location is `/var/www/html/sugar/`. Everything within `./data/app/` can be accessed through the browser, but the Sugar instance files have to be within `./data/app/sugar/`
* MySQL files are located in `./data/mysql/57/`
* For Elasticsearch 6.2 files are located in `./data/elasticsearch/62/`. For Elasticsearch 5.6 files are located in `./data/elasticsearch/56/` and so on.
* Redis files are located in `./data/redis/`
* LDAP files are located in `./data/ldap/`

Do not change the permissions of the various data subdirectories, as it might cause the system to not work correctly.

#### Sugar single instance application files - important notes
This setup is designed to run only a single Sugar instance. It also requires the application files to be exactly on the right place for the following three reasons:
1. File system permissions settings
2. PHP OPcache settings (eg: blacklisting of files that should not be cached)
3. Cronjob background process running

For the above reasons the single instance Sugar's files have to be located inside `./data/app/sugar/` (without subdirectories), for the stack setup to be working as expected.
If you do need multiple instances (eg: a Sugar version 8 and a version 7.9), as long as they are not running at the same time, you can leverage the provided tools to replicate and swap the data directories.

## Tips
### Utilities
To help with development, there are a set of tools within the `utilities` directory of the repository.
#### copysystem.sh
```./utilities/copysystem.sh data_80_clean data_80_clean_copy```
```
Copying "data_80_clean" to "data_80_clean_copy"
Copying data_80_clean to data_80_clean_copy
Copy completed, you can now swap or start the system
```
It helps to replicate a full `data_80_clean` content to another backup directory of choice (`data_80_clean_copy`). It requires the stack to be off (and it will check for it)
#### swapsystems.sh
```./utilities/swapsystems.sh backup_2018_06_28 data_80_clean```
```
Moving "data" to "backup_2018_06_28" and "data_80_clean" to "data"
Moving data to backup_2018_06_28
Moving data_80_clean to data
You can now start the system with the content of data_80_clean
```
It helps to move the current `data` directory to `backup_2018_06_28` and then `data_80_clean` to `data`, effectively swapping the current data in use. It requires the stack to be off (and it will check for it)
#### runcli.sh
```./utilities/runcli.sh php ./bin/sugarcrm password:weak```
It helps to execute a command within the CLI container. It requires the stack to be on

### Detect web server PHP error logs
To be able to achieve this consistently, it is recommended to leverage the single web server stack.
By running the command `docker logs -f sugar-web1` it is then possible to tail the output from the access and error log of Apache and/or PHP

### Fix Sugar permissions
You would just need to run again the permissions docker container with `docker start sugar-permissions`. The container will fix the permissions and ownership of files for you and then terminate its execution.
Apache and Cron run as the `sugar` user. Se the following options on `config_override.php`

```
$sugar_config['default_permissions']['user'] = 'sugar';
$sugar_config['default_permissions']['group'] = 'sugar';
```

### Run the included command line Repair
The application contains few scripts built to facilitate the system's repair from command line. The scripts will wipe the various caches (including OPcache and Redis if used). It will also warm-up as much as possible the application, to improve the browser experience on first load. The cron container from which the repair runs, has also been optimised to speed up the repairing processing.
To run the repair from the docker host, assuming that the repository has been checked out on sugardocker execute:

```
cd sugardocker
./repair
```

### Simplify stack startup and shutdown
Once you choose the most commonly used stack for the job, you could simply create two bash scripts to start/stop your cluster. Examples of how those could look like are below:

Start (eg: ~/8up):
```
#!/bin/bash
cd ~/sugardocker
docker-compose -f stacks/sugar8/php71.yml up -d
```

Stop (eg: ~/8down):
```
#!/bin/bash
cd ~/sugardocker
docker-compose -f stacks/sugar8/php71.yml down
```

Making sure that the bash script is executable with `chmod +x <script>`.

### Setup Sugar instance to leverage Redis object caching
Add on `config_override.php` the following options:
```
$sugar_config['external_cache_disabled'] = false;
$sugar_config['external_cache_disabled_redis'] = false;
$sugar_config['external_cache']['redis']['host'] = 'sugar-redis';
```
Make sure there are no other caching mechanism enabled on your config/config_override.php combination, otherwise set them as disabled = true.

### Run command line command or script
To run a PHP script execute something like the following sample commands:
```
docker@docker:~/sugardocker$ ./utilities/runcli.sh php ../repair.php --instance .
Debug: Entering directory .
Repairing...
Completed in 8 seconds
```

```
docker@docker:~/sugardocker$ ./utilities/runcli.sh whoami
sugar
```

```
docker@docker:~/sugardocker$ ./utilities/runcli.sh pwd
/var/www/html/sugar
```

If needed, sudo is available as well without the need of entering a password. Just make sure the permissions and ownership (user `sugar`) is respected.

### XHProf / Tideways profiling data collection

XHProf extension is configured on PHP 5.6 stacks, while Tideways extension is configured on PHP 7.1 stacks.

To enable profiling:
* Add [this custom code](https://gist.github.com/esimonetti/4c84541d49ee0828b31de91d30bcedb0) into your Sugar installation and repair the system (only if leveraging Tideways). Please note that the custom code does not have a namespace which is **intentional**. Adding a namespace will cause the profiling implementation to not find the TidewaysProf class.
* Configure `config_override.php` specific settings (see below based on the stack extension)

XHProf Sugar `config_override.php` configuration:
```
$sugar_config['xhprof_config']['enable'] = true;
$sugar_config['xhprof_config']['log_to'] = '../profiling';
$sugar_config['xhprof_config']['sample_rate'] = 1;
$sugar_config['xhprof_config']['flags'] = 0;
```

Tideways Sugar `config_override.php` configuration:
```
$sugar_config['xhprof_config']['enable'] = true;
$sugar_config['xhprof_config']['manager'] = 'TidewaysProf';
$sugar_config['xhprof_config']['log_to'] = '../profiling';
$sugar_config['xhprof_config']['sample_rate'] = 1;
$sugar_config['xhprof_config']['flags'] = 0;
```

Make sure new files are created on `./data/app/profiling/` when navigating Sugar. If not, ensure that the directory permissions are set correctly so that the `sugar` user can write on the directory.

Please note that profiling degrades user performance, as the system is constantly writing to disk profiling information and tracking application stats. Use profiling only on replica of the production environment.

### XHProf / Tideways profiling data analysis

* Download [XHProf viewer](https://github.com/sugarcrm/xhprof-viewer/releases/latest) zip file
* Unzip its files content within `./data/app/performance/`
* Make sure the `config_override.php` settings available on `./data/app/performance/` are kept as is (`<?php
$config['profile_files_dir'] = '../profiling';`)
* Access the viewer on http://docker.local/performance/ and verify that the collected data is viewable

### Typical Sugar config_override.php options for real-life development
```
$sugar_config['external_cache_disabled'] = false;
$sugar_config['external_cache_disabled_redis'] = false;
$sugar_config['external_cache_force_backend'] = 'redis';
$sugar_config['external_cache']['redis']['host'] = 'sugar-redis';
$sugar_config['external_cache_disabled_wincache'] = true;
$sugar_config['external_cache_disabled_db'] = true;
$sugar_config['external_cache_disabled_smash'] = true;
$sugar_config['external_cache_disabled_apc'] = true;
$sugar_config['external_cache_disabled_zend'] = true;
$sugar_config['external_cache_disabled_memcache'] = true;
$sugar_config['external_cache_disabled_memcached'] = true;
$sugar_config['cache_expire_timeout'] = 600; // default: 5 minutes, increased to 10 minutes
$sugar_config['disable_vcr'] = true; // bwc module only
$sugar_config['disable_count_query'] = true; // bwc module only
$sugar_config['save_query'] = 'populate_only'; // bwc module only
$sugar_config['collapse_subpanels'] = true; // 7.6.0.0+
$sugar_config['hide_subpanels'] = true; // bwc module only
$sugar_config['hide_subpanels_on_login'] = true; // bwc module only
$sugar_config['logger']['level'] = 'fatal';
$sugar_config['logger']['file']['maxSize'] = '10MB';
$sugar_config['developerMode'] = false;
$sugar_config['dump_slow_queries'] = false;
$sugar_config['import_max_records_total_limit'] = '2000';
$sugar_config['verify_client_ip'] = false;
```
Tweak the above settings based on your specific needs.

## Mac users notes
These stacks have been built on a Mac platform, that is known to not perform well with [Docker mounted volumes](https://github.com/docker/for-mac/issues/77).
Personally I run Docker on a Debian based minimal VirtualBox VM with fixed IP, running a NFS server. I either mount NFS on my Mac when needed or SSH directly into the VM. [The Debian Docker VirtualBox VM for Mac is available here](https://github.com/esimonetti/DebianDockerMac) with its [latest downloadable version here](https://github.com/esimonetti/DebianDockerMac/releases/latest).
