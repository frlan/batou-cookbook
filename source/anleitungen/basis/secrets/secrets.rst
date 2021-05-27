Secrets
=======

Eine Herausforderung bei Deplyoments ist es immer, dass Passwörter, Zertifikate und andere sensiblen Daten nicht unverschlüsselt in zum Beispiel einem Versionsverwaltungssystem abgelegt werden. Auf der anderen Seite, benötigt man diese beim tatsächlichen Materialisieren der Konfiguration auf dem Server, damit die Dienste wie gewünscht funktionieren. Batou bietet dazu die Möglichkeit, solche Informationen in den `secrets` zu hinterlegen -- mit dem OpenPGP-Standard verschlüsselte Dateien.

Dateien in den secrets ablegen
------------------------------

Hat man komplexere Strukturen mit geheimen Daten, wird der Ansatz einzelne Datenstrukturen in den Secrets abzulegen unter Umständen umständlich oder verhindert gar ein sinnvolles Management.

.. Warning::

    Batou unterstützt aktuell nur das Ablegen von Text-Dateien in den Secrets. Unter `Binaere_dateien_in_den_secrets`_ gibt es eine Anleitung, wie man das Problem lösen könnte.


Beispiel JSON
*************

Nehmen wir eine relative komplexe Konfigurationdatei im JSON-Format an:


.. code-block:: json

   {
       "password_server1": "mein_sehr_sicheres_passwort",
       "password_server2": "mein_sehr_sicheres_passwort",
       "password_server3": "mein_sehr_sicheres_passwort",
       "password_server4": "mein_sehr_sicheres_passwort",
       "password_server5": "mein_sehr_sicheres_passwort"
   }


Man könnte dieses Konfiguration auch über diesen Batou-Code abbilden:

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

Alternativ, unter der Bedinung, dass man die Passwörter der einzelnen Server innerhalb des Deployments nicht seperat nocheinmal benötigt, kann man die komplette JSON-Datei auch direkt sicher in die Secrets einchecken und einfach auf dem Server ausrollen

Zuerst legen wir die gewünschte Datei in den Secrets ab:

.. code-block:: sh

    $ ./batou secrets edit production myserverpasswords.json


Dadurch öffnet batou einen Editor und man kann den Inhalt einfügen. Beim Speichern und Schließen verschlüsselt batou diese Daten dann mit den Keys, die in der jeweiligen .cfg für die Umgebung angegeben sind. Hier in diesem Fall also ``secrets/production.cfg``.

Später kann man über ein Dict im Produktionsenvironment auf den Inhalt zugreifen:

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

t.b.p.s.
