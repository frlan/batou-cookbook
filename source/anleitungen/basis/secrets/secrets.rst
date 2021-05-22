Secrets
=======

Eine Herausforderung bei Deplyoments ist es immer, dass Passwörter, Zertifikate und andere sensiblen Daten nicht unverschlüsselt in zum Beispiel einem Versionsverwaltungssystem abgelegt werden. Auf der anderen Seite, benötigt man diese beim tatsächlichen materialisieren der Konfiguration auf dem Server, damit die Dienste wie gewünscht funktionieren. Batou bietet dazu die Möglichkeit, solche Informationen in den `secrets` zu hinterlegen -- mit dem OpenPGP-Standard verschlüsselte Dateien.

Generelle Verwendung
--------------------

batou secrets edit
******************

.. code-block:: console

    usage: batou secrets edit [-h] [--editor EDITOR] environment

    positional arguments:
      environment           Environment to edit secrets for.

    optional arguments:
      -h, --help            show this help message and exit
      --editor EDITOR, -e EDITOR
                            Invoke EDITOR to edit (default: $EDITOR or vi)

batou secrets summary
*********************

Mit dem ``summary``-Befehl kann man sich einen Überblick verschaffen, welcher Key Zugriff auf verschlüsselte Daten einer/aller Umgebung(en) hat.


.. code-block:: console

    usage: batou secrets summary [-h]

    optional arguments:
      -h, --help  show this help message and exit

Example:

.. code-block:: console

  $ ./batou secrets summary

  production
     members
      - alice@example.com
     secret files
      - secrets.yaml

  tutorial
     members
      - alice@example.com
      - bob@example.com
     secret files
      (none)


batou secrets add
*****************

.. code-block:: console

    usage: batou secrets add [-h] [--environments ENVIRONMENTS] keyid

    positional arguments:
      keyid                 The user's key ID or email address

    optional arguments:
      -h, --help            show this help message and exit
      --environments ENVIRONMENTS
                            The environments to update. Update all if not
                            specified.

batou secrets remove
********************

.. code-block:: console

    usage: batou secrets remove [-h] [--environments ENVIRONMENTS] keyid

    positional arguments:
      keyid                 The user's key ID or email address

    optional arguments:
      -h, --help            show this help message and exit
      --environments ENVIRONMENTS
                            The environments to update. Update all if not
                            specified.



Dateien in den secrets ablegen
------------------------------

Hat man komplexere Strukturen mit geheimen Daten, wird der Ansatz einzelne Datenstrukturen in den Secrets abzulegen unter Umständen umständlich oder verhindert gar ein sinnvolles Management.
.. Warning::

    Batou unterstützt aktuell nur das Ablegen von Text-Dateien in den Secrets. Unter `Binaere_dateien_in_den_secrets`_ gibt es eine Anleitung, wie man das Pröblem lösen könnte.



Beispiel JSON
*************

Nehmen wir eine relative komplexe Konfigurationdatei im JSON-Format an


.. code-block:: json

   {
       "password_server1": "mein_sehr_sicheres_passwort",
       "password_server2": "mein_sehr_sicheres_passwort",
       "password_server3": "mein_sehr_sicheres_passwort",
       "password_server4": "mein_sehr_sicheres_passwort",
       "password_server5": "mein_sehr_sicheres_passwort"
   }


Man könnte dieses Konfiguration auch über diesen Batou-Code abbilden

.. code-block:: python

    from batou.component import Component
    from batou.component import Attribute
    from batou.lib.file import JSONContent


    class MyServerKonfig(Component)

        password_server1 = Attribute(str)
        password_server2 = Attribute(str)
        password_server3 = Attribute(str)
        password_server4 = Attribute(str)
        password_server5 = Attribute(str)

        def configure(self):

            cnf = {
                "password_server1": self.password_server1,
                "password_server2": self.password_server2,
                "password_server3": self.password_server3,
                "password_server4": self.password_server4,
                "password_server5": self.password_server5}

            self += JSONContent(
                'myserverpasswords',
                data=cnf)

Jetzt nur noch für jede Umgebung password_server1 … password_server5 in den secrets definieren …

Alternativ, unter der Bedinung, dass man die Passwörter der einzelnen Server innerhalb des Deployments nicht seperat nocheinmal benötigt, kann man die komplette JSON-DAtei auch direkt sicher in die Secrets einchecken und einfach auf dem Server ausrollen

Zuerst legen wir die gewünschte Datei in den Secrets ab:

.. code-block:: sh

    $ ./batou secrets edit production myserverpasswords.json


Dadurch öffnet batou einen Editor und man kann den Inhalt einfügen. Beim Speichern und Schließen verschlüsselt batou diese Daten dann mit den Keys, die in der jeweiligen .cfg für die Umgebung angegeben sind. Hier in diesem Fall also ``secrets/production.cfg``.

Später kann man über ein Dict im Produktionsenvironment auf den Inhalt zugrefien

.. code-block:: python

    from batou.component import Component
    from batou.lib.file import File


    class MyServerKonfig(Component)

        def configure(self):

            self += File(
                'myserverpasswords',
                data=self.environment.secret_files['myserverpasswords.json'],
                sensitive_data=True)

Und da es sich dabei auch um vertrauliche Daten handelt, die im Log einer Pipeline nicht geloggt werden sollten, setzt dieses Beispiel noch das sensitive_data-Flag der File-Komponente.



Beispiel PEM
************
