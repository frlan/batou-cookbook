PostgreSQL
==========

Nutzer verwalten
----------------

Datenbanknutzer können einfach über ``batou_ext.postgres.User`` angelegt werden.

.. code-block:: python

    from batou_ext.postgres import User
    from batou import Component

    class PostgreSQLDB(Component)

        def configure(self):
            self += batou_ext.postgres.User(
                'crocodile',
                password='aligator3')


Datenbanken anlegen
-------------------

Eine Datenbank in einem PostgreSQL-Cluster kann einfach über ``batou_ext.postgres.DB`` angelegt werden:

.. code-block:: python

    from batou_ext.postgres import User
    from batou import Component


    class PostgreSQLDB(Component)

        def configure(self):
            self += batou_ext.postgres.DB(
                'my_datatase',
                owner='crocodile')

Wichtig ist es, den ``owner`` explizit zu setzen. Bei Bedarf kann auch das Encoding an dieser Stelle gesetzt werden:

.. code-block:: python

    from batou_ext.postgres import User
    from batou import Component


    class PostgreSQLDB(Component)

        def configure(self):
            self += batou_ext.postgres.DB(
                'my_datatase',
                owner='crocodile',
                locale='de_DE.UTF-8')

Der Standard hier ist ``en_US.UTF-8``.


Extensions installieren
-----------------------

PostgreSQL bietet die Möglichkeit, Extensions zu installieren.

Typischerweise besteht der Prozess mehreren Schritten:

#. Installieren der Erweiterung auf Betriebssystemebe über z. B. ``apt install poostgresql-13-postgis-3``
#. Aktivieren der Erweiterung für die konkrete Datenbank

Über ``batou_ext.postgres`` kann die Aktivierung sehr einfach gelöst werden

.. code-block:: python

    from batou_ext.postgres import Extension
    from batou import Component

    class PostgreSQLDB(Component)

        def configure(self):
            self += batou_ext.postgres.Extension(
                'uuid-ossp',
                db='my_database')


Datenbankserver läuft nicht als postgres-User
---------------------------------------------

Ein nicht sonderlich häufig vorkommendes Problem -- wenn dann doch sehr störend. Standardmäßig gehen die Komponenten aus ``batou_ext.postgres`` davon aus, dass der Cluster im Kontext der Users ``postgres`` läuft und so alle Befehle wie ``createdb``, ``psql`` via ``sudo -u postgres`` ausgefühtt werden können und damit die nötigen Berechtigungen erhalten. Sollte dem nicht der Fall sein, kann man dies auch einfach durch das Seten von ``command_prefix`` anpassen -- für dieses Beispiel nutzen wir den Nutzer ``pgsqluser``:

.. code-block:: python

    from batou.component import Attribute
    from batou.component import Component
    from batou_ext.postgres import Extension, DB, User


    class MyDataBase(Component)

        command_prefix = 'sudo -u pgsqluser')
        dbname = Attribute(str, 'my_database')
        dbuser = Attribute(str, 'crocodile')
        dbuserpassword = Attribute(str, 'aligator3')

        def configure(self):
            self += User(
                self.dbuser,
                password=self.dbuserpassword,
                command_prefix=self.command_prefix)

            self += DB(
                self.dbname,
                owner=self.dbuser,
                command_prefix=self.command_prefix)

            self += xtension(
                'uuid-ossp',
                db=self.dbname,
                command_prefix=self.command_prefix)

