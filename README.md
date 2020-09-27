# Project template for wagtail development

## Purpose

This repository contains a quick start (which I have repeated many times!) for an environment supporting a Django/Wagtail project, specifically the non-code infrastructure to build an environment for development with Wagtail. It is not dependent on any specific version of Django or Wagtail - it defaults to the "latest" release of Wagtail and installs a compatible version of Django.


## Features
* **!new!** starts with a fresh template from .env-template by default
* variable data is managed in a generated .env, which then become defaults for the next run, adjust just what you need
* `docker-compose-services.yml` for building and running all the stuff that’s (almost) always the same: PostgreSQL relational database for the ORM and redis nosql key store for web and session cache
* `docker-compose.yml` for running these with the containerised application

docker-compose.yml by default mounts the application directory on the host into the app container along with media and static folders, which is a suitable configuration for development. No development tools are installed within the container and it assumes that wheels exist for all packages, which is not true for some time after a new Python release.

For production use, volumes need to be modified in `docker-compose.yml`:

* remove the mount to the `${DJANGO_ROOT}` dir - the application code is already copied there in Dockerfile
* replace `${PWD}/${APP_NAME}` with `data-static` and `data-media` respectively to create and use internal docker volumes for data that need persistence. Alternatively these can be mounted anywhere on the host system.
* using a front-end or reverse proxy server (nginx or apache-http) would be required to expose correct ports and serve via https. `/media/` and `/static/` paths can be served directly from the front-end.
* uncomment `./manage.py collectstatic --no-input` in the Dockerfile and rebuild the app container `docker-compose build app`


## Pre-requisites

* git
* python 3.7+
* virtualenv (or python -m venv)
* (optional, but recommended) virtualenvwrapper
* bourne compatible shell (bash works)
* an internet connection
* docker, any recent version should work
* docker-compose, usually comes with the docker package but can be separately installed

#### Optional
* svn (recommended only to export this code instead of clone) - see below<br/>_Note: "git svn" has no **export** subcommand_.


## Setup Scripts

This is a three step process detailed in the next section:

*This repository can be used as a template directly on GitHub to generate a new project!*. To do so, click on the **Use this template** button on the [wagtail_init home page](https://github.com/deeprave/wagtail_init), clone your new repo and skip step 1.

1. export or clone this repository
2. run the **init.py** script (requires parameters for customisation, -h for help)
	* installs wagtail, Django and python dependencies
3. run the **initdb.sh** script (again, -h for help):
	* creates the database, role and user
	* does the initial database migration
	* creates the Django/Wagtail superuser
	* Builds the app container

Note: `init.sh` is now replaced with `init.py`, and also replaces `wagtail_settings.py`.

### Quick Start

The **Use this template** as detailed above is the fastest and easier way to get started.
Otherwise pulling down the git repository is unnecessary (unless you plan on making a pull request). If convenient, use **svn** to *export only the required the files* or use
```sh
$ svn export https://github.com/deeprave/wagtail_init/trunk targetdir
```
*targetdir* must be a direct subdirectory of the current directory for everything to work correctly.
If you don’t have svn installed, **git clone** and remove the pointless repository:
```sh
$ git clone https://github.com/deeprave/wagtail_init.git <targetdir>
$ rm -rf <targetdir>/.git
```
Change into your target dir, create or activate your python virtual environment and run the following script.


### init.py

Initial setup is done using `./init.py` (or `python init.py`). Variable data used by the docker-compose is saved in
 the file `.env` which caches data from previous runs. If init.py has never been run before, it will read data from
 `.env-template` instead.

Before starting, a python virtual environment needs to be created and activated - the script will complain if you don’t.
 If for some reason you want to install into the global environment (not recommended) you can skip this check by using
 the `--force`

Recommended but not required - create the virtual environment in some convenient place outside of the directory where
 you have exported or cloned this template. This is a sensible default if using virtualenvwrapper which takes care of 
 this cleanly and makes switching between virtual environments very simple. Also recommended - start with a clean 
 virtual environment. Reusing the same one over multiple projects is almost as bad as installing everything globally.

Run `./init.py -h` for help (shown below). This must be run from the the current folder. The django/wagtail project is
 created 1 level below in the named "app subdir".
```sh
usage: init.py [-h] [--project PROJECT] [--app APP] [--directory DIRECTORY] [--srvroot SRVROOT] [--novirtualenv NOVIRTUALENV] [--url URL] [--force] [--docker] [--volumes] [--randdb] [--randsa] [--secret] [--define [DEFINE]] [--mode {dev,beta,production,test}] [--prefix {1,2,3,4,5,6}] [--dbhost DBHOST]
               [--dbport DBPORT] [--dbname DBNAME] [--dbrole DBROLE] [--dbuser DBUSER] [--dbpass DBPASS] [--sapass SAPASS] [--rdhost RDHOST] [--rdport RDPORT] [--cache {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}] [--session {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}]
               [command [command ...]]

Wagtail project setup utility

positional arguments:
  command               (optional) command

optional arguments:
  -h, --help            show this help message and exit
  --project PROJECT, -X PROJECT
                        set project name ()
  --app APP, -a APP     set app name ()
  --directory DIRECTORY, -d DIRECTORY
                        set app directory (same as app name)
  --srvroot SRVROOT, -x SRVROOT
                        absolute root path for deployment
  --novirtualenv NOVIRTUALENV, -N NOVIRTUALENV
                        ignore virtualenv presence check
  --url URL, -U URL     set site base url (http://example.com)
  --force, -f           force wagtail project create
  --docker, -S          assume docker services
  --volumes, -v         use data volumes for app
  --randdb, -r          generate new random db user password
  --randsa, -R          generate new random db sa password
  --secret, -K          generate random SECRET_KEY
  --define [DEFINE], -D [DEFINE]
                        define one or more arguments
  --mode {dev,beta,production,test}, -m {dev,beta,production,test}
                        select django environment
  --prefix {1,2,3,4,5,6}, -E {1,2,3,4,5,6}
                        set an offset for std pg,redis ports

PostgreSQL:
  database options

  --dbhost DBHOST, -i DBHOST
                        database hostname (prefer IP)
  --dbport DBPORT, -p DBPORT
                        database port
  --dbname DBNAME, -n DBNAME
                        database or schema name
  --dbrole DBROLE, -g DBROLE
                        database role or owner
  --dbuser DBUSER, -u DBUSER
                        database username
  --dbpass DBPASS, -w DBPASS
                        database password
  --sapass SAPASS, -G SAPASS
                        database administrator password

Redis:
  cache options

  --rdhost RDHOST, -I RDHOST
                        redis hostname (prefer IP)
  --rdport RDPORT, -P RDPORT
                        redis port
  --cache {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}, -c {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}
                        redis cache db # (0-15)
  --session {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}, -s {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}
                        redis session db # (0-15)
```
The recommended usage is **`./init.sh [options] <app_name>`**. The "app_name" 
 provided is used to create a set of defaults, and otherwise uses reasonable
 defaults for most other values and allowing specific values to be overridden.
 Other options that should be used:

* `-R` generates random database passwords for both postgres and the application user.
* `-S` generates a random SECRET_KEY that is used by django.
* `-U <url>` sets the BASE_URL used by wagtail.
* Use -s <number> and/or -c <number> to offset port numbers by adding a prefix. This avoids port collisions with other 
  docker instances on the same host.

The content of ".env" is displayed. Hit ENTER  to continue.

At this point, the basic structure and requirements are set up and almost ready to run the development server.


### initdb.sh

This is the next step - `./initdb.sh` creates and sets up the PostgreSQL database.
```
initdb.sh: [options]
Options:
  -h    this help message
  -S    services run in docker (docker-compose-services)
  -r    reset postgres data
  -R    reset all docker data volumes for this project
  -b    (re)build the app docker container
  -g    update a local git repository (initialise if necessary)
  -m    run database migrations
```
Please take care when using this script, and read the effect of each option carefully as it allows databases and
 docker volumes to be completely destroyed and rebuilt. "reset postgres data" means dropping the database itself and
 users, recreating them from scratch. "reset all docker data volumes" will remove every volume referenced in all the
 docker files for the current project so can potentially nuke your database if you run it out of docker.

If the script detects that the database is not running it will be started (else all that follows will fail). When
 these containers are spun up for the first time on a system, images will be downloaded from hub.docker.com.

When run more than once for the same project and the postgres password has changed, the `-r` should be used to reset
 the previous database instance so that the postgres (system administrator) password can be set. This this can only
 be done when the database is initialised and will cause loss of any existing data. This is a destructive option and
 should not be used if the database contains anything of value. By default, a *password is required for the postgres
 user* by this docker image (the latest official postgres image is used), although there are workarounds for this by
 trusting any connection as detailed on the [postgres docker hub page](https://hub.docker.com/_/postgres), but this 
 is not recommended and not compatible with these scripts.


## Notes
### Database Setup

Using a role with all the required privileges and nologin is good practice when it comes to database user setup.

The login user - **NOT postgres**  - used by the application simply inherits all it needs from its parent role. This
 role is also given createdb privilege to enable creating and destroying the test database when running unit tests.
