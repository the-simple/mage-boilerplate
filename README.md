# Boilerplate for magento development

This project was build in order to help developer(s) to build magento environment based on CentOS.

It includes:
- nginx (openresty)
- php 7
- percona server
- redis
- centralized log (REK stack)
  - rsyslog
  - elasticsearch
  - kibana

## Installation and Run
First you need to clone the project

```bash
$ git clone git@github.com:the-simple/mage-boilerplate.git my-project
$ cd my-project
$ git submodule init
$ git submodule update
$ docker-compose up -d
```
Open http://localhost/ in you favorite browser.

## Magento instalation
as you can see in docker-compose.yml file, there is a volume `./var/www/html/magento` folder. Install Magento file into that folder and follow general magento installation steps.

### database hostname
when magent ask you database host you need to type there simple `db`
```xml
<!-- app/etc/local.xml -->
<default_setup>
  <connection>
    <host><![CDATA[db]]></host>
    <username><![CDATA[magento]]></username>
    <password><![CDATA[1qaz2wsx]]></password>
    <dbname><![CDATA[magento]]></dbname>
    <initStatements><![CDATA[SET NAMES utf8]]></initStatements>
    <model><![CDATA[mysql4]]></model>
    <type><![CDATA[pdo_mysql]]></type>
    <pdoType><![CDATA[]]></pdoType>
    <active>1</active>
  </connection>
</default_setup>
```
### redis cache
in order to setup cache you need to use `redis` as hostname
```xml
<!-- app/etc/local.xml -->
<cache>
  <backend>Cm_Cache_Backend_Redis</backend>
  <backend_options>
    <server>redis</server>
    <port>6379</port>
    <persistent></persistent>
    <database>2</database>
    <password></password>
    <force_standalone>0</force_standalone>
    <connect_retries>1</connect_retries>
    <read_timeout>10</read_timeout>
    <automatic_cleaning_factor>0</automatic_cleaning_factor>
    <compress_data>1</compress_data>
    <compress_tags>1</compress_tags>
    <compress_threshold>20480</compress_threshold>
    <compression_lib>gzip</compression_lib>
    <use_lua>1</use_lua>
  </backend_options>
</cache>
```

### Sample data
In order to get magento instalation with sample data you can use init sql folder for mysql service.

Uncommend line in db service where is `# - ./var/mysql-init/:/docker-entrypoint-initdb.d/`
```yml
volumes:
  - ./var/mysql/:/var/lib/mysql
 - ./var/mysql-init/:/docker-entrypoint-initdb.d/
```
And then put sql files to `./var/mysql-init` folder. Bear in mind that this should be done before first start of docker-compose project(it means before mysql is set up).

If you have already installed mysql service, you can run regular `docker exec`:
```bash
docker exec -it mageboilerplate_db_1 mysql -uroot -h127.0.0.1 magento < path/to/dump.sql
```

## Using local Dockerfiles
If you prefer build docker images from the source you can use Dockerfiles from `./src/`(submodules) folder.
Be sure that each service has uncommented `build` and commented `image` options
```yaml
services:
  db:
    # image: thesimple/centos-percona:57
    build: ./src/centos-percona
```

## Using external images
If you don't what or have no chance to build local images you can use prebuilt ones.
For that you need some modufications. For each service you need to uncomment `image` option and comment `build` option. For example:
```yaml
services:
  db:
    image: thesimple/centos-percona
    # build: ./src/percona
```

## Issues Tracker
Please, send any issues or questions to [Issue Tracker](https://github.com/the-simple/mage-boilerplate/issues) on github.

## Known issues

Elasticsearch doesn't start
`ERROR: bootstrap checks failed max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`

Solution:
```bash
$ sudo sysctl -w vm.max_map_count=262144
```

## TODO
- filter syslog data - use only required
- use syslog to track exception.log and system.log files from magento

## License
MIT License
