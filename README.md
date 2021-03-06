Wildfly 8 - CentOS Docker image
========================================

This repository contains the sources and
[Dockerfile](https://github.com/openshift/wildfly-8-centos/blob/master/Dockerfile)
of the base image for deploying JEE applications as reproducible Docker
images. The resulting images can be run either by [Docker](http://docker.io)
or using [geard](https://github.com/openshift/geard/).

Installation
---------------

This image is available as trusted build in [Docker Index](https://index.docker.io):

[index.docker.io/u/openshift/wildfly-8-centos](https://index.docker.io/u/openshift/wildfly-8-centos/)

You can install it using:

```
$ docker pull openshift/wildfly-8-centos
```

Repository organization
------------------------

* **`.sti/bin/`**

  This folder contains scripts that are run by [STI](https://github.com/openshift/geard/tree/master/sti):

  *   **assemble**

      Is used to restore the build artifacts from the previous built (in case of
      'incremental build'), to install the sources into location from where the
      application will be run and prepare the application for deployment (eg.
      installing maven dependencies, building java code, etc..).  
      
      In addition, the assemble script will copy artifacts provided in the 
      application source project into the Wildfly installation:
      
      Wildfly configuration files from the <application source>/cfg are copied 
      into the wildfly configuration directory.

      Pre-built war files from the <application source>/deployments are copied 
      into the wildfly deployment directory.
      
      Wildfly modules from the <application source>/provided_modules are copied 
      into the wildfly modules directory.
            

  *   **run**

      This script is responsible for running the application, by using the
      Wildfly application server.

  *   **save-artifacts**

      In order to do an *incremental build* (iow. re-use the build artifacts
      from an already built image in a new image), this script is responsible for
      archiving those. In this image, this script will archive the
      maven dependencies and previously built java class files.


  *   **test**

      After the application image is built, we want to check that when we create a new
      container based on that image, this container will respond to HTTP calls.
      This image contains a sample JEE application in the **test-app** folder. This
      sample application is run when a test is performed.

* **`wfbin/`**

  Contains script used to launch wildfly after performing environment variable
  substitution into the standalone.xml configuration file.

* **`wfcfg/`**
  Contains the default standalone.xml configuration which can be overriden by applications
  that provide a standalone.xml in <application_src>/cfg.
  
* **`wfmodules/`**  
  Contains commonly used modules such as postgres and mysql database drivers.
  
  
Environment variables
---------------------

*  **STI_SCRIPTS_URL** (default: '[.sti/bin](https://raw.githubusercontent.com/openshift/wildfly-8-centos/master/.sti/bin)')

    This variable specifies the location of directory, where *assemble*, *run* and
    *save-artifacts* scripts are downloaded/copied from. By default the scripts
    in this repository will be used, but users can provide an alternative
    location and run their own scripts.

Usage
---------------------

**Building the [openshift-jee-sample](https://github.com/bparees/openshift-jee-sample) JEE application..**

1. **using standalone [STI](https://github.com/openshift/geard/tree/master/sti) and running the resulting image by [Docker](http://docker.io):**
    
    ```
$ sti build git://github.com/bparees/openshift-jee-sample openshift/wildfly-8-centos wildflytest
$ docker run -p 8080:8080 wildflytest
```
    
2. **using `gear build` and running the resulting image as a systemd unit via [geard](https://github.com/openshift/geard/):**
    
    ```
$ gear build git://github.com/bparees/openshift-jee-sample openshift/wildfly-8-centos wildflytest
$ gear install wildflytest wildflytest-1 -p 8080:8080
$ gear start wildflytest-1
$ gear list-units
```

**Accessing the application:**
```
$ curl 127.0.0.1:8080
```

Copyright
--------------------

Released under the Apache License 2.0. See the [LICENSE](https://github.com/openshift/wildfly-8-centos/blob/master/LICENSE) file.
