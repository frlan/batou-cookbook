Zertifikate ausrollen
=====================


Eigenes PEM-Zertifikat ausrollen
--------------------------------

Ein typischer Fall ist, dass ein (PEM)-Zertifikate-Paar ausgerollt werden soll. War dies früher in den Zeiten vor Letsencrypt vor allem notwendig, um Server mit SSL/TLS zu sichern, werden solche Zertifikate heute noch weiter für Wildcard-Zertifikate oder für Authentifizierung von Anwendungen und Nutzern genutzt.


Der quick'n'dirty Weg
*********************

Im Prinzip ist eine PEM-Datei eine in BASE64 kodierte Textdatei, so dass diese mit Batous ``File``-Komponente ausgerollt werden könnte.

.. warning::
    Dieses Code-Beispiel geht davon aus, dass die Zertifikate unverschlüsselt(!) im Deployment-Repository im Verzeichnis der Komponente vorliegen, was natürlich ein sehr großes Sicherheitsproblem ist. Entsprechend sollte es nur verwendet werden, wenn der Zugriff auf den Quellcode anderweitig zuverlässig gesichert ist (also eigentlich gar nicht).

.. code-block:: python

    from batou.lib.file import File
    from batou.component import Component


    class MyCert(Component):

        def configure(self):

            self.privkey = File('privkey.pem')
            self.publickey = File('publickey.pem')
            self += self.privkey
            self += self.publickey



Zertifkat in den Secrets
************************

Um die Möglichkeit des missbräuchlichen Zugriffs einzugrenzen, empfiehlt es sich das Zertifikat in den Secrets des Deployments abzulegen und dann auf den eigentlichen Inhalt über eine Variable zuzugreifen.

.. code-block:: python

    from batou.lib.file import File
    from batou.component import Component
    from batou.component import Attribute


    class MyCert(Component):

        privkey_content = Attribute(str)
        publickey_content = Attribute(str)

        def configure(self):
            self.privkey = File('privkey.pem', content=self.privkey_content)
            self.publickey = File('publickey.pem',content=self.publickey_content)
            self += self.privkey
            self += self.publickey


Ein definiertes Zertifkatepaar mit batou_ext.ssl.Certificate() ausrollen
------------------------------------------------------------------------

.. hint::
	Diese Methode lohnt sich meist nur, wenn man a) auf verschiedenen Umgebugnen zwischen LetsEncrypt und einem anderen Zertifikat wechseln möchte und/oder b) das implizite Monitoring aus der ``Certificate``-Komponente nutzen möchte. In der Preaxis ist aber zumiest eine gute Idee, dies zu tun.

t.b.p.s.

LetsEncrypt mit mit batou_ext.ssl.Certificate()
-----------------------------------------------

.. warning::

    Einzelne Plattformen bieten Automatismen zum Generieren und Einbinden von LetsEncrypt-Zertifikaten an.

t.b.p.s.
