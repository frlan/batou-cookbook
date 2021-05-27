Textdateien
###########

Textdateien der verschiedensten Couleur sind wahrscheinlich der häufigste Fall, wenn es darum geht, Konfigurationsdateien oder andere Inhalte zu verteilen. Batou bietet mit der ``File``-Komponente eine sehr mächtige Komponente, die sehr viele Variationen zulässt.

Eine umfängliche Dokumentation der Optionen der Komponente kann man in der `Batou-Dokumentation <https://batou.readthedocs.io/en/latest/components/files.html>`_  finden.


Minimaler Aufruf
----------------

.. code-block:: python

    from batou.lib.file import File
    from batou.component import Component


    class MyApplication(Component):

        self += batou.lib.file.File('meineDatei.txt')

Batou sucht im Verzeichnis der Komponente nach einer Datei ``meineDatei.txt`` und legt sie auf dem Zielsystem unter ``~/deployment/work/myapplication/meineDatei.txt`` ab. Evtl. vorhandele `Jinja <https://jinja.palletsprojects.com/en/3.0.x/templates/>`_-Templating- Befehle werden dabei interpretiert.


Dateirechte
-----------

Besitzer & Gruppe & Zugriffsrechte
**********************************

Einzelne Programme benötigen definierte Owner und Dateiberechtigungen. Ein populäres Beispiel sind die Verzeichnisse und die Keys für OpenPGP.

.. code-block:: python

    from batou.lib.file import File
    from batou.component import Component


    class MyApplication(Component):

        self += batou.lib.file.File(
            'meineDatei.txt'
            owner='myuser',
            group='mygroup',
            mode=0o700)

``owner`` und ``group`` müssen als String übergeben werden. ``mode`` nimmt die gewünschten Rechte als Octalschreibweise an.

.. warning::

    Das explizite Einschränken von Dateirechten bzw. das Setzen von User und Group kann natürlich auch Gefahren bergen. ``mode=0o500`` könnte z. B. dazu führen, dass eine Datei nur einmal geschrieben werden kann. Hier ist also Vorsicht geboten.


Sensitive Daten
---------------

.. code-block:: python

    from batou.lib.file import File
    from batou.component import Component


    class MyApplication(Component):

        self += batou.lib.file.File(
            'meineDatei.txt'
            owner='myuser',
            group='mygroup',
            mode=0o700,
            sensitive_data=True)

Für Passwörter und andere "Geheimnisse" gibt es das Flag ``sensitive_data``, durch welches die entsprechende Datei explizit von z. B. der Anzeige des Diffs ausgenommen wird:

.. code-block::

    myvm123 > MyApplication > File('meineDatei.txt') > Content('meineDatei.txt')
    Not showing diff as it contains sensitive data.

Das ist sinnvoll, wenn das Deployment z. B. von einer CI/CD-Plattform ausgeführt werden soll, so dass Passwörter dort nicht im Log aufgetragen werden.
