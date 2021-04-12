.. _cite_test_guide:

Cite Test Guide
===============

A step by step guide to the GeoServer Compliance Interoperability Test Engine (CITE).

.. contents::

~~~~~~~~~~~~~


Check out CITE suite tests
--------------------------

.. note:: The CITE suite tests are available at `Open Geospatial Consortium`_.
.. _Open Geospatial Consortium: https://github.com/opengeospatial

Requirements:
-------------

- `GeoServer instance <https://github.com/geosolutions-it/geoserver>`_.

- `Teamengine Web Application <https://github.com/geosolutions-it/teamengine-docker>`_, with a set of CITE suite tests.

- make


CITE automation tests with docker
=================================


How to run the CITE Test suites with
`docker <https://www.docker.com>`_.

Requirements:
-------------

- Running the tests requires a linux system with `docker <https://www.docker.com>`_, `docker-compose <https://docs.docker.com/compose/install>`_, and git installed on it.

.. note::

   The CITE tools are available in the build/cite folder of the `GeoServer Git repository <https://github.com/geoserver/geoserver/tree/master/build/cite>`_:

Steps:
------

Set-up the environment.
~~~~~~~~~~~~~~~~~~~~~~~

   #.  Clone the repository.

       .. code:: shell

          git clone https://github.com/geosolutions-it/geoserver.git

   #.  go the cite directory.

       .. code:: shell

          cd geoserver/build/cite

   #.  inside will find a structure like below with a list of directories with the name of the suites to run.

       .. code:: shell

          cite
          |-- forms
          |-- geoserver
          |-- run-test.sh
          |-- wcs10
          |-- wcs11
          |-- wfs10
          |-- wms11
          |-- wms13
          |-- wfs11
          |-- logs
          |-- docker-compose.yml
          |-- postgres
          |-- README.md
          `-- Makefile

Running the suite tests.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   There is 2 way to run the suites, one is running with `make` that will
   automate all the commands, and the second one is running the test through WebUI:

   1. Running the through Makefile:

      -  run ``make`` in the console, will give you the list of commands
         to run.

         .. code:: shell

            make

      -  the output will like this:

         .. code:: makefile

            clean: $(suite)         Will Clean the Environment of previous runs.
            build: $(suite)         Will Build the GeoServer Docker Image for the Environment.
            test: $(suite)      Will running the Suite test with teamengine.

      - Choose which test to run by setting the Suite environment variable:

        .. warning::

            The first Docker build may take a long time.

        .. code:: SHELL

           suite=wcs10

        .. note::

           Valid values for the Suite variable are
             * wcs10
             * wcs11
             * wfs10
             * wfs11
             * wms11
             * wms13

      - Choose which GeoServer war to test by setting the ``war_url`` environment variable inside the ``Makefile``, ex:

        .. code:: C

          war_url = "https://build.geoserver.org/geoserver/main/geoserver-main-latest-war.zip"

      .. note::

        if you don't want to do it inside the ``Makefile`` you have the option of add the variable in the command when you build the docker images.

      -  To clean the local environment.

         .. code:: shell

            make clean suite=<suite-name>

      -  To build the geoserver docker image locally.

         .. code:: shell

            make build suite=<suite-name>

      - Alternative, with the ``war_url`` variable include:

         .. code::

           make build suite=<suite-name> war_url=<url-or-the-geoserver-war-file-desired>

      -  To run the suite test.

         .. code:: shell

            make test suite=<suite-name>

      -  And the last, but no less important run the full automate
         workflow.



         .. code:: shell

            make clean build test suite=<suite-name>


How to run TEAM Engine as single docker container.
---------------------------------------------------

- To run a standalone version of TEAM Engine, start it with the following command:

  .. code:: SHELL

     docker run -d --name standalone_teamengine -p 8080:8080 geosolutionsit/teamengine:latest

- TEAM Engine will be accessible on http://localhost:8080/teamengine/

- If you want to change the port, for example to have it on port "9090", change the command as follows:

  .. code:: SHELL

     docker run -d --name standalone_teamengine -p 9090:8080 geosolutionsit/teamengine:latest

- To stop TEAM Engine:

  .. code:: SHELL

     docker stop standalone_teamengine

Run CITE Test Suites in local pc
================================

.. note::

   I assume that you have an standalone geoserver running.


Requirements:
-------------

- GeoServer running.

- PostgreSQL with PostGIS extension installed. (only for the WFS Tests Suites)

- Teamengine Running in docker container.

- `GeoServer repository <https://github.com/geoserver/geoserver.git>`_


#. Clone the repository:

   .. code:: shell

      git clone https://github.com/geoserver/geoserver.git

#. Change directory to the ``cite``

   .. code:: shell

      cd geoserver/build/cite

#. Check the commands available:

   - Run ``make`` to check:

   .. code:: shell

        make


   - you should get an output as following:

   .. code:: makefile

        clean: $(suite)		 Will Clean the Environment of previous runs.
        build: $(suite)		 Will Build the GeoServer Docker Image for the Environment.
        test: $(suite)		 Will running the Suite test with teamengine.
        webUI: $(suite)		 Will running the Suite test with teamengine.


Run WFS 1.0 tests
-----------------

.. important::

   Running WFS 1.0 tests require PostgreSQL with PostGIS extension installed to be installed on the system.

Requirements:
~~~~~~~~~~~~~

- `GeoServer running`
- teamengine
- Posgresql
- PostGIS

#. Prepare the environment:

   - login in postgresql and create a user named "cite".

   .. code:: sql

     createuser cite;

   - Create a database named "cite", owned by the "cite" user:

   .. code:: sql

     createdb cite own by cite;

   - enter to the database and enable the postgis extension:

   .. code:: sql

    create extension postgis;

   - Change directory to the citewfs-1.0 data directory and execute the script cite_data_postgis2.sql:

   .. code-block:: shell

    cd <root of geoserver repository>
    psql -U cite cite < cite_data_postgis2.sql

   - Start GeoServer with the citewfs-1.0 data directory. Example:

   .. important::

     If the postgresql server is not in the same host of the geoserver, you have to change the `<entry key="host">localhost</entry>` in the `datastore.xml` file, located inside each workspace directory. ex.

     .. note::

       <root of geoserver sources>/data/citewfs-1.0/workspaces/cgf/cgf/datastore.xml

   .. code-block:: shell

    cd <root of geoserver install>
    export GEOSERVER_DATA_DIR=<root of geoserver sources>/data/citewfs-1.0
    ./bin/startup.sh

#. Start the test:

   .. code:: shell

     make webUI

#. Go to the browser and open the URL: http://localhost:8888/teamengine/

   - after the site open, click on the **Sign in** button and enter the user and password. 

   With the following parameters:

   #. ``Capabilities URL`` http://<ip.of.the.goserver>:8080/geoserver/wfs?request=getcapabilities&service=wfs&version=1.0.0

   #. ``Enable tests with multiple namespaces`` tests included

      .. image:: ./image/tewfs-1_0.png

Run WFS 1.1 tests
-----------------

.. important::

   Running WFS 1.1 tests require PostgreSQL with PostGIS extension installed to be installed on the system.

Requirements:
~~~~~~~~~~~~~
- GeoServer
- teamengine
- Posgresql
- PostGIS

#. Prepare the environment:

   - login in postgresql and create a user named "cite".

   .. code:: sql

     createuser cite;

   - Create a database named "cite", owned by the "cite" user:

   .. code:: sql

     createdb cite own by cite;

   - enter to the database and enable the postgis extension:

   .. code:: sql

    create extension postgis;

   - Change directory to the citewfs-1.1 data directory and execute the script dataset-sf0-postgis2.sql:

   .. code-block:: shell

    cd <root of geoserver repository>
    psql -U cite cite < dataset-sf0-postgis2.sql

   - Start GeoServer with the citewfs-1.1 data directory. Example:

   .. important::

     If the postgresql server is not in the same host of the geoserver, you have to change the `<entry key="host">localhost</entry>` in the `datastore.xml` file, located inside each workspace directory. ex.

     .. note::

       <root of geoserver sources>/data/citewfs-1.1/workspaces/cgf/cgf/datastore.xml

   .. code-block:: shell

    cd <root of geoserver install>
    export GEOSERVER_DATA_DIR=<root of geoserver sources>/data/citewfs-1.1
    ./bin/startup.sh


#. Start the test:

   .. code:: shell

     make webUI

#. Go to the browser and open the URL: http://localhost:8888/teamengine/

   - after the site open, click on the **Sign in** button and enter the user and password.

   .. note:: the Default username/password are **teamengine/teamengine**.

   With the following parameters:

   #. ``Capabilities URL`` http://<ip.of.the.goserver>:8080/geoserver/wfs?service=wfs&request=getcapabilities&version=1.1.0

   #. ``Supported Conformance Classes``:

      * Ensure ``WFS-Transaction`` is *checked*
      * Ensure ``WFS-Xlink`` is *unchecked*

   #. ``GML Simple Features``: ``SF-0``

   .. image:: ./image/tewfs-1_1.png

Run WMS 1.1 tests
-----------------

#. Prepare the environment:

  - Start GeoServer with the citewms-1.1 data directory. Example:

   .. code-block:: shell

    cd <root of geoserver install>
    export GEOSERVER_DATA_DIR=<root of geoserver sources>/data/citewms-1.1
    ./bin/startup.sh

#. Start the test:

   .. code:: shell

     make webUI

#. Go to the browser and open the URL: http://localhost:8888/teamengine/

   - after the site open, click on the **Sign in** button and enter the user and password.

   .. note:: the Default username/password are **teamengine/teamengine**.

   With the following parameters:

   #. ``Capabilities URL``

          http://<ip.of.the.geoserver>:8080/geoserver/wms?service=wms&request=getcapabilities&version=1.1.1

   #. ``UpdateSequence Values``:

      * Ensure ``Automatic`` is selected
      * "2" for ``value that is lexically higher``
      * "0" for ``value that is lexically lower``

   #. ``Certification Profile`` : ``QUERYABLE``

   #. ``Optional Tests``:

      * Ensure ``Recommendation Support`` is *checked*
      * Ensure ``GML FeatureInfo`` is *checked*
      * Ensure ``Fees and Access Constraints`` is *checked*
      * For ``BoundingBox Constraints`` ensure ``Either`` is selected

   #. Click ``OK``

   .. image:: ./image/tewms-1_1a.png

   .. image:: ./image/tewms-1_1b.png

Run WCS 1.0 tests
-----------------

#. Prepare the environment:

  - Start GeoServer with the citewcs-1.0 data directory. Example:

   .. code-block:: shell

    cd <root of geoserver install>
    export GEOSERVER_DATA_DIR=<root of geoserver sources>/data/citewcs-1.0
    ./bin/startup.sh

#. Start the test:

   .. code:: shell

     make webUI

#. Go to the browser and open the URL: http://localhost:8888/teamengine/

   - after the site open, click on the **Sign in** button and enter the user and password.

   .. note:: the Default username/password are **teamengine/teamengine**

   With the following parameters:

   #. ``Capabilities URL``:

          http://<ip.of.the.geoserver>:8080/geoserver/wcs?service=wcs&request=getcapabilities&version=1.0.0

   #. ``MIME Header Setup``: "image/tiff"

   #. ``Update Sequence Values``:

      * "2" for ``value that is lexically higher``
      * "0" for ``value that is lexically lower``

   #. ``Grid Resolutions``:

      * "0.1" for ``RESX``
      * "0.1" for ``RESY``

   #. ``Options``:

      * Ensure ``Verify that the server supports XML encoding`` is *checked*
      * Ensure ``Verify that the server supports range set axis`` is *checked*

   #. ``Schemas``:

      * Ensure that ``The server implements the original schemas from the WCS 1.0.0 scpecification (OGC 03-065`` is selected

   #. Click ``OK``

   .. image:: ./image/tewcs-1_0a.png

   .. image:: ./image/tewcs-1_0b.png

   .. image:: ./image/tewcs-1_0c.png


Run WCS 1.1 tests
-----------------

#. Prepare the environment:

  - Start GeoServer with the citewcs-1.1 data directory. Example:

   .. code-block:: shell

    cd <root of geoserver install>
    export GEOSERVER_DATA_DIR=<root of geoserver sources>/data/citewcs-1.1
    ./bin/startup.sh


#. Start the test:

   .. code:: shell

     make webUI

#. Go to the browser and open the URL: http://localhost:8888/teamengine/

   - after the site open, click on the **Sign in** button and enter the user and password.

   .. note:: the Default username/password are **teamengine/teamengine**

   With the following parameters:

   #. ``Capabilities URL``:

         http://<ip.of.the.geoserver>:8080/geoserver/wcs?service=wcs&request=getcapabilities&version=1.1.1

   Click ``Next``

   .. image:: ./image/tewcs-1_1a.png



.. _commandline:

.. _teamengine:
