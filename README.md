# osixia/phpldapadmin

[![Docker Pulls](https://img.shields.io/docker/pulls/osixia/phpldapadmin.svg)][hub]
[![Docker Stars](https://img.shields.io/docker/stars/osixia/phpldapadmin.svg)][hub]
[![Image Size](https://img.shields.io/imagelayers/image-size/osixia/phpldapadmin/latest.svg)](https://imagelayers.io/?images=osixia/phpldapadmin:latest)
[![Image Layers](https://img.shields.io/imagelayers/layers/osixia/phpldapadmin/latest.svg)](https://imagelayers.io/?images=osixia/phpldapadmin:latest)

[hub]: https://hub.docker.com/r/osixia/phpldapadmin/

Latest release: 0.6.8 - phpLDAPadlin 1.2.3 (with php5.5 patch) - [Changelog](CHANGELOG.md) | [Docker Hub](https://hub.docker.com/r/osixia/phpldapadmin/) 

A docker image to run phpLDAPadmin.
> [phpldapadmin.sourceforge.net](http://phpldapadmin.sourceforge.net)

- [Quick start](#quick-start)
	- [OpenLDAP & phpLDAPadmin in 1'](#openldap--phpldapadmin-in-1)
- [Beginner Guide](#beginner-guide)
	- [Use your own phpLDAPadmin config](#use-your-own-phpldapadmin-config)
	- [HTTPS](#https)
		- [Use autogenerated certificate](#use-autogenerated-certificate)
		- [Use your own certificate](#use-your-own-certificate)
		- [Disable HTTPS](#disable-https)
	- [Fix docker mounted file problems](#fix-docker-mounted-file-problems)
	- [Debug](#debug)
- [Environment Variables](#environment-variables)
	- [Set your own environment variables](#set-your-own environment-variables)
		- [Use command line argument](#use-command-line-argument)
		- [Link environment file](#link-environment-file)
		- [Make your own image or extend this image](#make-your-own image-or-extend-this-image)
- [Advanced User Guide](#advanced-user-guide)
	- [Extend osixia/phpldapadmin:0.6.8 image](#extend-osixiaphpldapadmin068-image)
	- [Make your own phpLDAPadmin image](#make-your-own-phpldapadmin-image)
	- [Tests](#tests)
	- [Kubernetes](#kubernetes)
	- [Under the hood: osixia/web-baseimage](#under-the-hood-osixiaweb-baseimage)
- [Changelog](#changelog)

## Quick start

Run a phpLDAPadmin docker image by replacing `ldap.example.com` with your ldap host or IP :

    docker run -p 6443:443 \
           --env PHPLDAPADMIN_LDAP_HOSTS=ldap.example.com \
           --detach osixia/phpldapadmin:0.6.8

That's it :) you can access phpLDAPadmin on [https://localhost:6443](https://localhost:6443)

### OpenLDAP & phpLDAPadmin in 1'

Example script:

    #!/bin/bash -e
		docker run --name ldap-service --hostname ldap-service --detach osixia/openldap:1.1.1

		docker run --name phpldapadmin-service --hostname phpldapadmin-service --link ldap-service:ldap-host --env PHPLDAPADMIN_LDAP_HOSTS=ldap-host --detach osixia/phpldapadmin:0.6.8

		PHPLDAP_IP=$(docker inspect -f "{{ .NetworkSettings.IPAddress }}" phpldapadmin-service)

    echo "Go to: https://$PHPLDAP_IP"
    echo "Login DN: cn=admin,dc=example,dc=org"
    echo "Password: admin"


## Beginner Guide

### Use your own phpLDAPadmin config
This image comes with a phpLDAPadmin config.php file that can be easily customized via environment variables for a quick bootstrap,
but setting your own config.php is possible. 2 options:

- Link your config file at run time to `/container/service/phpldapadmin/assets/config.php` :

      docker run --volume /data/my-config.php:/container/service/phpldapadmin/assets/config.php --detach osixia/phpldapadmin:0.6.8

- Add your config file by extending or cloning this image, please refer to the [Advanced User Guide](#advanced-user-guide)

### HTTPS

#### Use autogenerated certificate
By default HTTPS is enable, a certificate is created with the container hostname (it can be set by docker run --hostname option eg: phpldapadmin.my-company.com).

	docker run --hostname phpldapadmin.my-company.com --detach osixia/phpldapadmin:0.6.8

#### Use your own certificate

You can set your custom certificate at run time, by mounting a directory containing those files to **/container/service/phpldapadmin/assets/apache2/certs** and adjust their name with the following environment variables:

	docker run --volume /path/to/certifates:/container/service/phpldapadmin/assets/apache2/certs \
	--env PHPLDAPADMIN_HTTPS_CRT_FILENAME=my-cert.crt \
	--env PHPLDAPADMIN_HTTPS_KEY_FILENAME=my-cert.key \
	--env PHPLDAPADMIN_HTTPS_CA_CRT_FILENAME=the-ca.crt \
	--detach osixia/phpldapadmin:0.6.8

Other solutions are available please refer to the [Advanced User Guide](#advanced-user-guide)

#### Disable HTTPS
Add --env PHPLDAPADMIN_HTTPS=false to the run command :

    docker run --env PHPLDAPADMIN_HTTPS=false --detach osixia/phpldapadmin:0.6.8

### Fix docker mounted file problems

You may have some problems with mounted files on some systems. The startup script try to make some file adjustment and fix files owner and permissions, this can result in multiple errors. See [Docker documentation](https://docs.docker.com/v1.4/userguide/dockervolumes/#mount-a-host-file-as-a-data-volume).

To fix that run the container with `--copy-service` argument :

		docker run [your options] osixia/phpldapadmin:0.6.8 --copy-service

### Debug

The container default log level is **info**.
Available levels are: `none`, `error`, `warning`, `info`, `debug` and `trace`.

Example command to run the container in `debug` mode:

	docker run --detach osixia/phpldapadmin:0.6.8 --loglevel debug

See all command line options:

	docker run osixia/phpldapadmin:0.6.8 --help

## Environment Variables

Environment variables defaults are set in **image/environment/default.yaml**

See how to [set your own environment variables](#set-your-own-environment-variables)

- **PHPLDAPADMIN_LDAP_HOSTS**: Set phpLDAPadmin server config. Defaults to :

  ```yaml
  - ldap.example.org:
    - server:
      - tls: true
    - login:
      - bind_id: cn=admin,dc=example,dc=org
  - ldap2.example.org
  - ldap3.example.org
  ```
  This will be converted in the phpldapadmin config.php file to :
  ```php5
  $servers->newServer('ldap_pla');
  $servers->setValue('server','name','ldap.example.org');
  $servers->setValue('server','host','ldap.example.org');
  $servers->setValue('server','tls',true);
  $servers->setValue('login','bind_id','cn=admin,dc=example,dc=org');
  $servers->newServer('ldap_pla');
  $servers->setValue('server','name','ldap2.example.org');
  $servers->setValue('server','host','ldap2.example.org');
  $servers->newServer('ldap_pla');
  $servers->setValue('server','name','ldap3.example.org');
  $servers->setValue('server','host','ldap3.example.org');
  ```
  All server configuration are available, just add the needed entries, for example:  
  ```yaml
  - ldap.example.org:
    - server:
      - tls: true
      - port: 636
      - force_may: array('uidNumber','gidNumber','sambaSID')
    - login:
      - bind_id: cn=admin,dc=example,dc=org
      - bind_pass: p0p!
    - auto_number:
      - min: 1000
  - ldap2.example.org
  - ldap3.example.org
  ```

  See complete list: http://phpldapadmin.sourceforge.net/wiki/index.php/LDAP_server_definitions

  If you want to set this variable at docker run command add the tag `#PYTHON2BASH:` and convert the yaml in python:

		docker run --env PHPLDAPADMIN_LDAP_HOSTS="#PYTHON2BASH:[{'ldap.example.org': [{'server': [{'tls': True}]},{'login': [{'bind_id': 'cn=admin,dc=example,dc=org'}]}]}, 'ldap2.example.org', 'ldap3.example.org']" --detach osixia/phpldapadmin:0.6.8

  To convert yaml to python online: http://yaml-online-parser.appspot.com/

Apache :
- **PHPLDAPADMIN_SERVER_ADMIN**: Server admin email. Defaults to `webmaster@example.org`

HTTPS :
- **PHPLDAPADMIN_HTTPS**: Use apache ssl config. Defaults to `true`
- **PHPLDAPADMIN_HTTPS_CRT_FILENAME**: Apache ssl certificate filename. Defaults to `phpldapadmin.crt`
- **PHPLDAPADMIN_HTTPS_KEY_FILENAME**: Apache ssl certificate private key filename. Defaults to `phpldapadmin.key`
- **PHPLDAPADMIN_HTTPS_CA_CRT_FILENAME**: Apache ssl CA certificate filename. Defaults to `ca.crt`

Reverse proxy HTTPS :
- **PHPLDAPADMIN_TRUST_PROXY_SSL**: Set to `true` to trust X-Forwarded-Proto header

Ldap client TLS/LDAPS :

- **PHPLDAPADMIN_LDAP_CLIENT_TLS**: Enable ldap client tls config, ldap serveur certificate check and set client  certificate. Defaults to `true`
- **PHPLDAPADMIN_LDAP_CLIENT_TLS_REQCERT**: Set ldap.conf TLS_REQCERT. Defaults to `demand`
- **PHPLDAPADMIN_LDAP_CLIENT_TLS_CA_CRT_FILENAME**: Set ldap.conf TLS_CACERT to /container/service/ldap-client/assets/certs/$PHPLDAPADMIN_LDAP_CLIENT_TLS_CA_CRT_FILENAME. Defaults to `ldap-ca.crt`
- **PHPLDAPADMIN_LDAP_CLIENT_TLS_CRT_FILENAME**: Set .ldaprc TLS_CERT to /container/service/ldap-client/assets/certs/$PHPLDAPADMIN_LDAP_CLIENT_TLS_CRT_FILENAME. Defaults to `ldap-client.crt`
- **PHPLDAPADMIN_LDAP_CLIENT_TLS_KEY_FILENAME**: Set .ldaprc TLS_KEY to /container/service/ldap-client/assets/certs/$PHPLDAPADMIN_LDAP_CLIENT_TLS_KEY_FILENAME. Defaults to `ldap-client.key`

	More information at : http://www.openldap.org/doc/admin24/tls.html (16.2.2. Client Configuration)

Other environment variables:
- **PHPLDAPADMIN_CFSSL_PREFIX**: cfssl environment variables prefix. Defaults to `phpldapadmin`, cfssl-helper first search config from PHPLDAPADMIN_CFSSL_* variables, before CFSSL_* variables.
- **LDAP_CLIENT_CFSSL_PREFIX**: cfssl environment variables prefix. Defaults to `ldap`, cfssl-helper first search config from LDAP_CFSSL_* variables, before CFSSL_* variables.

### Set your own environment variables

#### Use command line argument
Environment variables can be set by adding the --env argument in the command line, for example:

	docker run --env PHPLDAPADMIN_LDAP_HOSTS="ldap.example.org" \
	--detach osixia/phpldapadmin:0.6.8

#### Link environment file

For example if your environment file is in :  /data/environment/my-env.yaml

	docker run --volume /data/environment/my-env.yaml:/container/environment/01-custom/env.yaml \
	--detach osixia/phpldapadmin:0.6.8

Take care to link your environment file to `/container/environment/XX-somedir` (with XX < 99 so they will be processed before default environment files) and not  directly to `/container/environment` because this directory contains predefined baseimage environment files to fix container environment (INITRD, LANG, LANGUAGE and LC_CTYPE).

#### Make your own image or extend this image

This is the best solution if you have a private registry. Please refer to the [Advanced User Guide](#advanced-user-guide) just below.

## Advanced User Guide

### Extend osixia/phpldapadmin:0.6.8 image

If you need to add your custom TLS certificate, bootstrap config or environment files the easiest way is to extends this image.

Dockerfile example:

    FROM osixia/phpldapadmin:0.6.8
    MAINTAINER Your Name <your@name.com>

    ADD https-certs /container/service/phpldapadmin/assets/apache2/certs
    ADD ldap-certs /container/service/ldap-client/assets/certs
    ADD my-config.php /container/service/phpldapadmin/assets/config.php
    ADD environment /container/environment/01-custom


### Make your own phpLDAPadmin image

Clone this project :

	git clone https://github.com/osixia/docker-phpLDAPadmin
	cd docker-phpLDAPadmin

Adapt Makefile, set your image NAME and VERSION, for example :

	NAME = osixia/phpldapadmin
	VERSION = 0.6.8

	becomes :
	NAME = billy-the-king/phpldapadmin
	VERSION = 0.1.0

Add your custom certificate, environment files, config.php ...

Build your image :

	make build

Run your image :

	docker run -d billy-the-king/phpldapadmin:0.1.0

### Tests

We use **Bats** (Bash Automated Testing System) to test this image:

> [https://github.com/sstephenson/bats](https://github.com/sstephenson/bats)

Install Bats, and in this project directory run :

	make test

### Kubernetes

Kubernetes is an open source system for managing containerized applications across multiple hosts, providing basic mechanisms for deployment, maintenance, and scaling of applications.

More information:
- http://kubernetes.io
- https://github.com/kubernetes/kubernetes

A kubernetes example is available in **example/kubernetes**

### Under the hood: osixia/web-baseimage

This image is based on osixia/web-baseimage.
More info: https://github.com/osixia/docker-web-baseimage

## Changelog

Please refer to: [CHANGELOG.md](CHANGELOG.md)
