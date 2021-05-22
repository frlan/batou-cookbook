Textdateien
###########

Textdateien der verschiedensten Colour sind wahrscheinlich der häufigste Fall, wenn es darum geht Konfigurationsdateien oder andere Inhalte zu verteilen. Batou bietet mit der File-Komponente eine sehr mächtige Komponente, die sehr viele Variationen zulässt.

Eine umfängliche Dokumentation der Optionen der Komponente kann man in der `Batou-Dokumentation <https://batou.readthedocs.io/en/latest/components/files.html>`_  finden.

.. code-block:: python

    import batou.lib.file

    self += batou.lib.file.File()


Dateirechte
-----------

Besitzer & Gruppe
*****************


Mode
****

Nicht sleten muss man eine Datei einen bestimmten Mode geben. Dazu bietet ``File()`` das Argument ``mode``, welches den gewünschten Mode in Octal-Schreibweise entgegennimmt:

.. code-block:: python

    import batou.lib.file

    self += batou.lib.file.File(
        'meineDatei',
        mode=0o700)

Hier wird eine Datei, die nur vom Besitzen gelesen, geschrieben oder ausgeführt werden kann, angelegt.

.. warning::

	Das explizite Einschränken von Dateirechten kann natürlich auch Gefahren bergen. ``mode=0o500`` köntte z. B. dazu führen, dass eine Datei nur einmal geschrieben werden kann.

Sensitive Daten
---------------


Für Passwörter und andere "Geheimnisse" gibt es das Flag ``sensitive_data``, durch welches die entsprechende Datei explizit von z. B. der Anzeige des diffs ausgenommen wird:

.. code-block::

    myvm123 > Backend > File('work/backend/dbcredentials.conf') > Content('dbcredentials.conf')
    Not showing diff as it contains sensitive data.


Das ist sinnvoll, wenn das Deployment z. B. von einer CI-Plattform ausgeführt werden soll, die

