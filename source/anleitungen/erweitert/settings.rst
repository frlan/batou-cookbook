Settings und Versionsauswahl
============================


Versionskonfiguration
---------------------

Ein typisches Problem bei komplexeren Deployments ist, dass man die verschiedenen Versionen der einzelnen Komponenten an einer Stelle konzentriert haben möchte.

Versionen über Standardwerte in der Komponente setzen
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

Das funktioniert zuverlässig und wenn man einen Entwicklungsworkflow mit verschiedenen Branches benutzt, können so die Versionsupdates zuverlässig durch mergen (main -> staging -> production) zuverlässig auf den einzelnen Umgebungen ausgerollt werden.

Versionen über die Umgebung steuern
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nehmen wir einmal dieses Beispiel, welches für eine Anwendung ein Backoffice, eine Frontend und ein Backend (aka API-Anwendungsserver) sehr minimalistisch definiert:

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

... und ebenfalls indivduell setzen. Das funktioniert soweit ganz gut, wenn man

Ansatz
