Settings und Versionsauswahl
============================

Settings
--------

Oft kommt es vor, dass man ein Datum an verschiedenen stellen eines Deployments benötigt. Ein sehr typisches Beispiel sind die ``allowed_hosts``-Settings bei einigen Frameworks, durch die sichergestellt wird, dass nur unter der richtigen Domain auf eine Anwendung zugegriffen werden kann. Man muss diesen Namen konseqequent im Deployment benutzen:

* Am Reverse-Proxy, um den Server-Name für z. B. den Nginx richtig zu setzen
* Bei der Verwwaltung der TLS-Zertifikate
* Im Backend (die Anwendung)
* …

Durch einfaches ``provide()`` und ``require()``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Meist ergibt sich aus der Struktur der Anwendung ein Baum, in dem man auf die einzelnen Daten zugreifen kann.

Ein einfaches Beispiel mit zwei Komponenten:

.. code-block:: python

        from batou.component import Component
        from batou.component import Attribute
        from batou.lib.file import File


        class Backend(Component):

            api_name = Attribute(str, 'api.example.org')

            def configure(self):

                self.provide('backend', self)


        class Webserver(Component):

                self.backend = self.require('backend')
                self += File('domain.txt', content=self.backend.api_name


Die Herausforderung bei einem solchen Design ist es, dass man die richtige Denkrichtung befolgt: Das Datum, welches man an vielen Stellen der Anwendung benötigt, sollte an einem Blatt des Abhängigkeitenbaum definiert werden. Hier in diesem Beispiel ist es deswegen innerhalb der ``Backend``-Komponente eingetragen und entsprechend zu konfigurieren. Nicht zwingend offensichtlich zu finden, wenn man an dem Deployment etwas tun möchte.


Durch ``provide()`` und ``require()`` … reversed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

t.b.p.s.

Durch eine zentrale Settings-Komponente
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Auch wenn die anderen Möglichkeiten durchaus ihren Anwendungsberiech haben und wahrscheinlich gut funktionieren, zeigte sich in der Praxis, dass die Schaffung einer zentrallen ``Settings``-Komponente für solche Fälle die Lösung mit den ringsten Kopfschmerzen war.

.. code-block::

    from batou.component import Component
    from batou.component import Attribute
    from batou.lib.file import File

    class Settings(Component):

        api_name = Attribute(str)
        www_name = Attribute(str)

        def configure(self):
            self.provide("settings", self)

    class Backend(Component):

        def configure(self):

            self.settings = self.require_one('settings')
            self += File('domain.txt', content=self.settings.www_name)


    class Frontend(Component):
        def configure(self):

            self.settings = self.require_one('settings')

            # Heranziehen des Backendes für z.B. das Docroot oder
            # der Adresse, auf der das Backend zu erreichen ist
            # In dem Beispiel ungenutzt.
            self.backend = self.require_one('backend')

            self += File('domain.txt', content=self.settings.www_name)



Versionskonfiguration
---------------------

Ein typisches Problem bei komplexeren Deployments ist, dass man die verschiedenen Versionen der einzelnen Komponenten an einer Stelle konzentriert haben möchte.

Versionen über die Komponente setzen
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Der einfachste Weg Versionen zu setzen, ist dies direkt an der Komponente zu tun.

.. code-block:: python

    from batou.component import Component
    from batou.component import Attribute
    from batou.lib.file import File


    class Backend(Component):

        def configure(self):

            self += File('version.conf', content='1.23')


    class Frontend(Component):

        def configure(self):

            self += File('version.conf', content='1.42')


    class BackOffice(Component):

        def configure(self):

            self += File('version.conf', content='3.1415')


Das funktioniert zuverlässig und wenn man einen Entwicklungsworkflow mit verschiedenen Branches benutzt, können so die Versionsupdates zuverlässig durch mergen (main -> staging -> production) zuverlässig auf den einzelnen Umgebungen ausgerollt werden. Der Nachteil liegt aber darin, dass man für jede Teilanwendung in den Dateibaum einsteigen muss und dort in der ``component.py`` den Wert setzen muss. Das ist gerade sehr aufwendig, wenn die Versionsupdates aus einer CI/CD-Lösung angestoßen werden sollen.


Versionen über die Umgebung steuern
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nehmen wir einmal ein Beispiel, welches für eine Anwendung ein Backoffice, ein Frontend und ein Backend (z. B. ein API-Anwendungsserver) sehr minimalistisch definiert:

.. code-block:: python

    from batou.component import Component
    from batou.component import Attribute
    from batou.lib.file import File


    class Backend(Component):

        version = Attribute(str)

        def configure(self):

            self += File('version.conf', content=self.version)


    class Frontend(Component):

        version = Attribute(str)

        def configure(self):

            self += File('version.conf', content=self.version)


    class BackOffice(Component):

        version = Attribute(str)

        def configure(self):

            self += File('version.conf', content=self.version)

Man kann jetzt die Versionen in den jeweiligen Umgebungen aussteuern:

.. code-block::

    [host:host01]
    components = backend, frontend, backoffice

    [component:backend]
    version = 1.23

    [component:frontend]
    version = 1.42

    [component:backoffice]
    version = 1.23

... und ebenfalls individuell setzen. Das funktioniert soweit ganz gut, wenn man den manuellen Aufwand nicht scheut, für jede Umgebung die Werte bei Bedarf explizit zu setzen.


Versionen über eine zentrale Konfigurationsdatei steuern
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Eine Möglichkeit ist es, die beiden vorhergehenden Ansätze zu vereinen und eine zentrale Konfigurationsdatei allein für Versionen einzuführen. Dies erlaubt es, einfach über eine Drittanwendung das Versionsupdate anzustoßen, dabei aber den Vorteil einer merge-basierenden Workflows zu verlieren.

Wir definieren eine ``versions.ini`` im Basis-Verzeichnisses des Deployments für alle drei Teile der Anwendung aus den vorhergegangen Beispielen:

.. code-block::

    [frontend]
    version = 1.42

    [backoffice]
    version = 1.23

    [backend]
    version = 1.23


Das ini-Format ist einfach zu bearbeiten, sowohl von Mensch als auch Maschine, und so gut wie jede Programmiersprache bietet eine sinnvolle Möglichkeit an, dass mit überschaubarem Aufwand zu machen.

.. hint::

    Im Prinzip kann man dafür auch YAML oder JSON benutzen. Die Herangehensweise ist damit die Gleiche und der Code muss entsprechend angepasst werden.

Um auf die Versionen zugreifen zu können, ist es sinnvoll eine generische Settings-Komponente einzuführen.

.. code-block:: python

    from batou.component import Component
    from batou.component import Attribute
    import configparser

    class Settings(Component)

        versions_ini = Attribute(str, 'versions.ini')

        def configure(self):
            self.provide("settings", self)
            self._load_versions()

        def _load_versions(self):
            self.versions = config = configparser.ConfigParser()
            self.provide('versions', self.versions)
            versions_ini = os.path.normpath(
                os.path.join(self.root.defdir, "..", "..", self.versions_ini))
            config.read(versions_ini)

``_load_versions()`` liest die ``versions.ini`` ein und stellt sie als eine Python-Struktur zur Verfügung. An dieser Stelle könnte man auch einfach eine JSON oder YAML oder XML oder .-Datei entsprechend parsen. Für die eigenltichen Komponeten der Anwendung (Backend, Backoffice, Frontend) wurde an dieser Stelle aber ein gemeinsames Interface geschaffen, über welches sie die konkretne Daten erhalten können.

.. code-block:: python

    from batou.component import Component
    from batou.component import Attribute
    from batou.lib.file import File


    class Backend(Component):

        def configure(self):
            self.settings = self.require_one('settings')

            self += File(
                'version.conf',
                content=self.settings.versions.get('backend', 'version'))


    class Frontend(Component):

        def configure(self):
            self.settings = self.require_one('settings')

            self += File(
                'version.conf',
                content=self.settings.versions.get('frontend', 'version'))


    class BackOffice(Component):

        def configure(self):
            self.settings = self.require_one('settings')

            self += File(
                'version.conf',
                content=self.settings.versions.get('backoffice', 'version')



.. hint::

    Nicht vergessen, die neue ``Settings()`` in die Umgebungen einzubinden:

    .. code-block::

        [host:host01]
        components = backend, frontend, backoffice, settings
