Building and Running the Intervention Engine Stack in a Containerized Environment with Docker (Linux)
========================================================================================================

This document details the steps needed to build and deploy the Intervention Engine stack in a containerized environment using [Docker](https://www.docker.com/). To deploy the Intervention Engine stack in a local development environment, please refer to [Building and Running the Intervention Engine Stack in a Development Environment](https://github.com/intervention-engine/ie/blob/master/docs/dev_install.md).

These instructions are written for deployment on linux operating systems and have been tested with Ubuntu Server version 14.x and RedHat Enterprise Linux 7.1. Some steps may vary for other operating systems. For instructions to deploy Intervention Engine with Docker on Mac OS X, please refer to the instructions found [here](https://github.com/intervention-engine/ie-docker/blob/master/docs/docker-mac.md)

Install Prerequisite Tools
==========================

Building and running Intervention Engine with Docker requires the following 3rd party tools:

- docker-engine
- docker-compose

Install Docker Engine
---------------------
[Docker Engine](https://www.docker.com/products/docker-engine) is the lightweight runtime that builds and runs your Docker containers, required for the creation and deployment of the Intervention Engine component containers.

To install the Docker Engine, follow the instructions found in the [Install Docker](https://docs.docker.com/linux/step_one/) documentation on the Docker website.

Install Docker Compose
----------------------

[Docker Compose](https://docs.docker.com/compose/overview/) is the Docker tool that orchestrates the linking of containers to allow them to communicate with one another.

To install Docker Compose, follow the instructions found in the [Install Docker Compose](https://docs.docker.com/compose/install/) documentation on the Docker website, starting with step 3.

Download the docker-ie Repository
=================================
The [docker-ie](https://github.com/intervention-engine/docker-ie) repository contains the _docker-compose.yml_ file that tells Docker what images to download and how to configure them.  It also contains initial configuration files that may be modified due to site-specific needs.

The easiest way to get a local copy of the docker-ie repository is to download it from GitHub using the following URL:
[https://github.com/intervention-engine/ie-docker/archive/master.zip](https://github.com/intervention-engine/ie-docker/archive/master.zip)

Then unzip the _master.zip_ file to a location of your choice.  The easiest option is to unzip it somewhere in your user folder (for example, _/home/jsmith/ie-docker_), since Docker for Mac automatically shares user folders with Docker containers.  If you unzip it to a different location, then you'll need to configure Docker for Mac to share that folder.

_NOTE: If you are comfortable with [Git](https://git-scm.com/), you may use Git tools to clone the ie-docker repository instead._

Configure ie-docker
===================
Docker Compose allows the orchestration and configuration of multiple Docker containers.  Docker Compose's main configuration information is contained in the appropriately named docker-compose.yml file. _This file should not need to be edited._  Most of the user-configurable settings are externalized to other files.

Configure ie-multifactorriskservice and ie-integrator Containers
----------------------------------------------------------------
The _ie-multifactorriskservice_ and _ie-integrator_ containers are site-specific.  In fact, they generally only apply to our current use case, where a specific HIE is used for integrating medical records and REDCap is used for integrating risk assessments.  Future guides will demonstrate how to run Intervention Engine _without_ these services.

The main configuration parameters for these services exist in the file _ie-docker/config/site.env_:
```
# The following variables are specific to each site where intervention engine is installed.
# These variables MUST be set to the correct values for your site!

# ie-multifactorriskservice variables
REDCAP_URL=http://myredcapserver/redcap/api
REDCAP_TOKEN=MYREDCAPTOKEN

# ie-integrator variables
HIE_URL=http://myhieserver/api
HIE_USER=myhieusername
HIE_PASSWORD=myhiepassword
```

Each line of this file either contains a comment (starting with `#`) or an environment variable assignment.  To change the value for an environment variable, edit the text directly after the `=` sign.  All of the environment variables starting with _REDCAP_ or _HIE_ must be set appropriately for the configuration to work.  When you've finished editing the values, save the file.

Indicate the Patients to Integrate Data For
-------------------------------------------
The _ie-integrator_ service needs a list of medical record numbers to indicate the patients that should have their data downloaded from the HIE and REDCap.  This list is contained in the file _ie-docker/config/ie-integrator/ee\_file.txt_.  Each line in the file should contain a separate medical record number, for example:
```
# list employee identifiers to import from the hie
# one identifier per line
173894729
038450281
384920297
```

Create or Configure SSL Certificates and Keys
=============================================
In order for nginx to use secure http (`https`), it requires an ssl certificate and key. These can be generated (self-signed certificate) or obtained from a Certificate Authority. To generate a certificate and key, navigate to the _ie-docker/config/ie-nginx_ folder and run the following command:

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx.key -out nginx.crt
```

You should then have the following files:
* _ie-docker/config/ie-nginx/nginx.key_
* _ie-docker/config/ie-nginx/nginx.cert_

If you obtained your certificate and key from a Certificate Authority, simply rename them to `nginx.key` and `nginx.cert` and move or copy them into the _ie-docker/config/ie-nginx_ folder.

Add Users to System
===================
To add users to Intervention Engine, you must use the `htpasswd` utility to append users to the _ie-docker/config/ie-nginx/htpasswd_ file.

RedHat Enterprise Linux 7.1
---------------------------

For RedHat Enterprise Linux, you must install _httpd-tools_, which contains the `htpasswd` utility.  This can be done using _yum_:

```
sudo yum install httpd-tools
```

Ubuntu Server 14.x
------------------

For Ubuntu Server, you must install _apache2-utils_, which contains the `htpasswd` utility.  This can be done using _apt-get_:

```
yum install httpd-tools
```

All Systems
-----------

Once the package containing `htpasswd` has been installed, navigate to the _ie-docker/config/ie-nginx_ folder and run the following command, replacing `exampleuser` with the username you would like to add:

```
sudo htpasswd htpasswd exampleuser
```

The tool will prompt you for a password:

```
New password:
Re-type new password:
Adding password for user exampleuser
```

This will add an entry to the `htpasswd` file with the username and encrypted password. _NOTE: Please keep in mind any site-specific security requirements when entering a password.  The htpasswd utility does not enforce password requirements, so it is up to you to choose appropriate passwords._

Repeat this step for each user you would like to add to the system.  Users can be added this way at any time.

Run docker-compose to Build and Launch the Containers
=====================================================
Once your prerequisite software is installed, and your site-specific configurations saved, you can use docker-compose to build and launch all of the Intervention Engine containers.

Navigate to the _ie-docker_ folder containing the _docker-compose.yml_ file and run the following command:

```
docker-compose up
```

Docker will then begin downloading and building containers. Once the containers are built, docker-compose will report all stdout output from the running containers, prepended by the container name. Please note that docker-compose will then be running in the foreground of your terminal session.

Once all of the containers are built and running, you can connect to intervention engine over https port 443 (e.g., https://myserver.mydomain.org/). Your browser should prompt you for a username and password when you connect.

Data and Logging
----------------

Assuming you have not changed the default parameters, running docker-compose will automatically create _data_ and _logs_ folders in your _ie-docker_ folder.  The _data_ folder will contain the actual database files for Intervention Engine, along with other important and pertinent data.  Do not remove, move, or edit these files -- they are crucial to Intervention Engine.  If you wish to backup Intervention Engine data, this is the folder to back up.  (Note that since this contains the database, it contains PHI).  The _logs_ folder contains relevant logs that may be helpful in debugging.

Updating Docker Images
======================

The current _docker-compose.yml_ configuration references the _latest_ versions of images.  That said, once docker-compose has a _latest_ image on disk, it does not necessarily poll for new images.  You can force docker-compose to pull down new images in order to ensure you are running the latest versions.

To begin, run the following command to stop the docker containers:

```
docker-compose down
```

This should produce an output similar to the following:

```
Stopping iedocker_ie-nginx_1 ... done
Stopping iedocker_ie-integrator_1 ... done
Stopping iedocker_ie-multifactorriskservice_1 ... done
Stopping iedocker_ie-ccda-endpoint_1 ... done
Stopping iedocker_ie-server_1 ... done
Stopping iedocker_mongo_1 ... done
Removing iedocker_ie-nginx_1 ... done
Removing iedocker_ie-integrator_1 ... done
Removing iedocker_ie-multifactorriskservice_1 ... done
Removing iedocker_ie-ccda-endpoint_1 ... done
Removing iedocker_ie-server_1 ... done
Removing iedocker_mongo_1 ... done
Removing network iedocker_default
```

At this point, you can update any stale images by issuing the following command:

```
docker-compose pull
```

This will download any new images, producing output related to the downloads as well as messages indicating when an image is already at the latest version.  Now you can start the services again by simply running:

```
docker-compose up
```