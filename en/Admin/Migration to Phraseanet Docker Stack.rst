Migration to Phraseanet Docker Stack
====================================

.. topic:: Requirements

    Phraseanet Docker stack up and running. For more information on how to install Phraseanet with Docker go to https://github.com/alchemy-fr/Phraseanet#phraseanet-with-docker.

    A dump of the application box and databoxes of the Phraseanet install you whish to migrate.

    A backup of the following diretories : custom, lazaret, download as well as the datas directories ("db_name_of_the_databox").

    A backup of your previous configuration file.


Migrate Lazaret , download , custom and datas directories
*********************************************************

Copy your lazaret, download, custom and datas directories directory to the new destination (on your Phraseanet fresh install, take note note of the path define inside your env.local file).


Importing application box and databoxes
***************************************

Import the ab and dbs to the mysql container using the following commands

Ab:

.. code-block:: bash

    docker exec -i <mysql_container_tag> mysql -uuser -ppass <ab_name_of_the_applicationbox_to_import> < <db_name_of_the_applicationbox_to_import>.sql

Dbs:

.. code-block:: bash

    docker exec -i <mysql_container_tag> mysql -uuser -ppass <db_name_of_the_databox_to_import> < <db_name_of_the_databox_to_import>.sql

Apply the changes to the newly imported ab and dbs to reflect the configuration inside your env.local:

On  the ‘Sbas’ table in the application box report the changes made inside the  env.local in accordance to the env variables:

.. code-block:: bash

    docker exec -i <mysql_container_tag> -uuser -ppass -e "USE <ab_name>; UPDATE sbas SET host='host', user='user', pwd='pwd';"

Change the storage path to reflect the paths defined inside your env.local on your dbs:

.. code-block:: bash
 
    docker exec -i <mysql_container_tag> mysql -uuser -ppass -e "USE <db_name_of_the_databox>; UPDATE subdef SET path=REPLACE(path,'<OLD_PATH>','<NEW_PATH>');"

.. code-block:: bash
 
    docker exec -i <mysql_container_tag> mysql -uuser -p -e "USE <db_name_of_the_databox>; UPDATE pref SET value=REPLACE(value,'<OLD_PATH>','<NEW_PATH>');"

Set the key inside configuration file
*************************************

Copy and pass the key from the older configuration.yml file inside the newly created configuration file:

.. code-block:: bash
 
    nano config/configuration.yml

.. code-block:: yaml

    main:
        key: mysecretkey

Then compile the configuration from the worker container:

.. code-block:: bash

    docker-compose -f docker-compose.yml run --rm worker bin/console comp:conf

Upgrade application 
*******************

Launch the “builder” container and plays the upgrade:

.. code-block:: bash
 
    docker-compose -f docker-compose.yml run --rm worker bin/setup system:upgrade

Launch the populate of the index
********************************

you can populate the inex using the builder container with:

.. code-block:: bash

    docker-compose -f docker-compose.yml run --rm worker bin/console searchengine:index -p
