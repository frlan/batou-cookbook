Häufige Fragen
##############

Eine Frage des Stil? Wie macht man das am Besten? Eine Sammlung von F.A.Q., die sonst nicht in eine andere Kategorie passen

Was muss ich in die Secrets schreiben?
--------------------------------------

In die Secrets gehören alle Informationen, die nicht öffentlich werden sollten. D.h. Zugangskennungen (Login, API-Keys, Passwörter, …) oder auch Zertifikate. Das sollte sich nicht nur auf externe Dienste oder Zugangsdaten beschränken, die einen Zugang zur Anwendung gewähren, sondern sich auch auf Passwörter für z. B. im Deployment verwaltete Datenbanken, Mail-Server, … ausweiten. Man sollte mit Kunden und Partnern das prinzipielle Installieren einer Anwendung anhand des Deployments besprechen können, ohne ihnen Zugriff auf sensible Daten zu geben. Auch wenn es aus anderen Gründen vielleicht nicht opportun ist, ist eine gute Regel des Daumens: Kann ich das Deployment so wie es ist bei github.com/gitlab.com hochladen, ohne das eine unberechtigte Person Zugang zu den Einzelsystemen erhält. (Bei vielen Standardanwendungen ist die Art sie zu installieren gut öffentlich dokumentiert.)


Standardwerte für Klassenvariablen
----------------------------------

Hier sind verschiedene Ansätze möglich, die sich mit den entgegengesetzten Zielen einfache Pflege und Sicherheit vor falscher Konfiguration unterschiedlich auseinandersetzen.

.. code-block:: python

    from batou.component import Attribute, Component

    class MeineKomponent(Componet):

        # Variante 1
        var1 = Attribute(str)

        # Variante 2
        var2 = Attribute(str, 'Testvalue')

        # Variante 3
        var3 = Attribute(str, 'Produktionssetting')

#. **Werte frei lassen**: Diese Version ist wahrscheinlich die sicherste Ansatz. Die Werte sind freigelassen und ``Attribute()`` stellt sicher, dass das Deployment nur durchgeführt werden kann, wenn ein (sinnvoller) Wert gesetzt sind. Dieser Ansatz sorgt aber auch dafür, dass alle und jede Option in den Umgebungsdateien explizit angegeben werden muss, was zu einem gehörigen Aufwand im Bereich der Buchhaltung führt.
#. **Werte auf die Entwicklungsumgebung setzen**: Dieser Ansatz ist in soweit sicher, als dass durch eine Fehlkonfiguration nicht z. B. Mails aus den Testumgebungen an Empfänger der Produktivumgebung versendet werden und eine Entwicklung in den Test-Systemen schnell vorangehen kann. Für das Live-Stellen eines Features benötigt es dann aber sehr sorgfältiges Prüfen der neuen Variablen, so dass diese in der Produktionsumgebung richtig über die Umgebungsdateien gesetzt werden können.
#. **Werte auf die Produktionseinstellungen setzen**: Der Gedanke dahinter ist, dass die Anzahl der explizit gesetzten Werte für die Produktionsumgebung möglichst gering sind, und so der Pflegeaufwand (für diese Umgebung) gering bleibt, wodurch ebenfalls die Fehleranfälligkeit sinkt.

In der Praxis hat sich gezeigt, dass ein Ansatz aus Variante 1 und 3 oftmals sinnvoll ist: Zugangsdaten wie Passwörter, API-Keys, Zertifikate nicht mit einem Standardwert versehen, dafür aber Variablen, die ein bestimmtes Verhalten einer Anwendung aussteuern (z. B. ``debug=True``) auf den Produktionssettings vordefiniert zu haben.

Kurze Aufrufe oder mit kompletten Modulpfad?
--------------------------------------------

Darauf gibt es keine richtige Antwort. Ob man nun

.. code-block:: python

    import batou.component
    import batou.lib.file


    class MeineKomponente(batou.component.Component):

        def configure(self):

            self += batou.lib.file.File('meineDatei')


oder

.. code-block:: python

    from batou.component import Component
    from batou.lib.file import File


    class MeineKomponente(Component):

        def configure(self):

            self += File('meineDatei')


ist vor allem eine Frage des Geschmackes. Die erste Version hat den
Vorteil, dass bei verschiedenen Quellen von ähnlich benannten
Komponenten klar ist, wo die Komponente herkommt (z. B. wenn batou_ext
benutzt), die zweite lässt sich einfacher lesen und schreiben, weil
wesentlich weniger Redundanzen im Textanteil vorliegen.


GPG UI oder Key-ID in den Secrets
---------------------------------

Man kann die Keys, welche für das Verschlüsseln der Secrets verwendet wrden, in zwei grundsätzlichen Arten angeben:

#. Über die Key-ID
#. Über eine UID eines Keys

Die UID ist dabei eine konkrete Emailadresse, die von GnuPG dann in einen konrketen Key umgewandelt wird. Die Key-ID ist der entweder kurze oder längere Fingerprint des Schlüssels.

Ein großer Vorteil der Methode über die UID ist, dass man sehr schnell erkennen kann, für wen die Secrets verschlüsselt sind. So gibt z. b. ``./batou secrets summary`` für den Fall eine Liste der Emailadressen aus. Der Nachteil ist, dass beim erneuten Verschlüsseln GnuPG einen Key für den Nutzer auswählt -- je nachdem, was es im Schlüsselbund findet und als vertrauenswürdig einstuft. Das kann dazu führen, dass die Secrets von jedem Nutzer mit einem effektiv anderen Set an Schlüsseln verschlüsselt werden.

Die Angabe über KeyID ist dabei wesentlich eindeutiger, aber nicht gut lesebar, da z. B. summary in dem Fall tatsächlich nur die Liste der KeyID ausgibt, welche dann z. B. über `gpg --finger`` für den Nutzer in ein Mapping auf einen Nutzer umgewandelt werden muss.

