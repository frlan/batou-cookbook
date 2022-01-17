Referenzierung von Komponenten
==============================

Batou kann Abhängigkeiten zwischen Komponenten abbilden. Das kann sinnvoll sein, wenn eine Komponenten ohne eine andere nicht funktioniert -- z. B. ist ein ``git clone`` eines externen Quellcode-Repositories ohne einen SSH-Key nicht möglich. Ein anderes Beispiel ist, wenn auf Daten einer anderen Komponente zugegriffen werden soll oder eine zeitliche Reihenfolge sichergestellt werden soll.


Provide und Require
-------------------

Zur einfachen Abbildung von Abhängigkeiten bietet eine Komponente die Methoden ``provide()``, sowie ``require()`` bzw. ``require_one()`` an.


.. code-block:: python

    from batou.component import Component

    class MyComponentProvide(Component):

        def configure(self):
            self.provide('mycomponent1', self)


    class MyComponentRequire(Component):

        def configure(self):
            self.comp1 = self.require_one('mycomponent1')
            # oder
            self.comp1x = self.require('mycomponent1')



`self.provide` stellt einen Wert, hier ``self``, die vollständig konfigurierte Komponente auf dem Zielsystem über einen Alias zur Verfügung -- `mycomponent1`. Über diesen kann an anderer Stelle genau auf das Objekt zugegriffen werden. Dies kann aber auch nur ein String-Objekt oder z. B. ein Tupel sein. In der Praxis propagiert man in sehr vielen Fällen das komplette Objekt.

Für den Zugriff gibt es zwei Möglichkeiten:
`require()` bzw. `require_one()` nehmen als ersten Parameter den Namen des Aliases entgegen und geben das entsprechende Objekt zurück. Dabei unterscheiden sie sich in der Verwendung,

Kann kein passendes Objekt gefunden werden, gibt es eine Fehlermeldung und das Deployment wird nicht gestartet: Offensichtlich ist das Deployment nicht konsistent.


``require()``
^^^^^^^^^^^^^

``require()`` gibt _alle_ im Deployment aktuell aktiven Objekte mit diesem Alias als eine Liste zurück. Dies kann sinnvoll sein, wenn man z. B. einen Loadbalancer mit verschiedenen Backends konfigurieren möchte:


.. code-block:: python

    from batou.component import Component

    class Backend(Component):

        def configure(self):
            self.provide('applicationserver', self)


    class Loadbalancer(Component):

        def configure(self):
            self.backends = self.require('mycomponent1')


Nun könnte man in einem Jinja-Template für die Konfiguration über die Backens iterieren

.. code-block:: python

        self += File('haproxy.cfg')


.. code-block::

        backend app
        {% for server in component.servers %}
        server {{server.host.name}} {{server.address.listen}} check inter 10s rise 2 fall 1 maxconn 2
        {% endfor %}


``require_one()``
^^^^^^^^^^^^^^^^^

``require_one()`` gibt exakt 1 Objekt zurück. Das ist immer dann sinnvoll, wenn man im Deployment ein, und nur ein Vorkommen einer bestimmten Komponente benötigt. So benötigt eine Webanwendung nur eine Instanz der Datenbankzugangsdaten oder nur eine Instanz mit Informationen zum Mailserver.

Sind im Deployment mehrer Instanzen des gesuchten Objektes vorhanden, kommt es zu einer Fehlermeldung.

Umgekehrte Beziehung
--------------------

In einigen Fällen ist es notwendig, dass man ein Abhängigkeitsverhältnis umgedreht werden muss. Hat man z. B. eine Situaiton, in der die Anwendung eine Datenbank benöttigt, die Datenbank aber nur mit Informationen aus der Anwendung definiert werden kann -- weil z B. der Datanbankname sich von einer Option der Awnendung ableitet, kann man die Richtung umdrehen


.. code-block:: python

        from batou.component import Component

        class Anwendung(Component):

            def configure(self):

                self.provide('application', self)

        class Database(Component):

            def configure(self):

                self.app = self.require_one(
                    'application', reverse=True)


So kann man in der Datenbank-Komponente auf Werte der Anwendung zugreifen, die inhaltliche Reihenfolge -- die Datenbank muss vor der Anwnedung angelegt werden -- bleibt aber bestehen.

Dies kann auch genutzt werden, um multige Abhängigkeiten zu definieren:

.. code-block:: python

        class MaintenancePage(Component):
            def configure(self):
                self.require("dbbackend", reverse=True)
                self.require("frontend", reverse=True)
                self.require("backend", reverse=True)


Sanfte Beziehungen
------------------

Hin und wieder kommt es vor, dass man keine harten Abhängigkeiten benötigt. Ist z. B. eine Teilkomponente optional, muss dann aber in der richtigen Reihenfolge ausgerollt werden oder entsprechend integriert werden, kann es durch das setzen von ``strict=False`` als solches gekennzeichnet werden. Batou wird dann bei einem Deployment nicht abbrechen, wenn diese Abhängigkeiten nicht erfüllt ist.

.. code-block:: python

    from batou.component import Component

    class MyComponentProvide(Component):

        def configure(self):
            self.provide('mycomponent1', self)


    class MyComponentRequire(Component):

        def configure(self):
            self.comp1 = self.require_one('mycomponent1', strict=False)


