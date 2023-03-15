Löschen von Dateien und Verzeichnissen
======================================

Batou beitet die Möglichkeit, sicherzustellen, dass bestimmte Verzeichnisse
und/oder Dateien nicht da sind. Das ist besonders nützlich, wenn man zum
Beispiel eine Migration eines Services macht und sicherstellen muss, dass eine
alte Konfiuration und eine neue Konfiguration nicht zur gleichen Zeit
existieren. 

Die Nutzung ist recht einfach:

.. code-block:: python
   
    from batou.lib.file import Purge 

    # …
    
        def configure(self):
            self += Purge("/datei/die/nicht/da/sein/soll")


``Purge()`` nimmt dabei ein ein Pattern, welches globbing unterstützt. Möchte man
also z. B. eine komplette Struktur löschen – sagen wir ein Verzeichnis mit
Unterverzeichnissen und Dateien darin – oder ein bestimmtes Muster von Dateien,
kann man dies über das typische globbing abbilden

.. code-block:: python
   
    from batou.lib.file import Purge 

    # …
    
        def configure(self):
            self += Purge("/etc/nginx/sites-enabled/old-*.conf")



