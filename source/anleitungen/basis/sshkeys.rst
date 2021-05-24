SSH-Keys
========

Zum Auschecken von Quellcode-Repositories auf dem Zielhost oder andere Tätigkeiten braucht man oft SSH-Keys. Das Deployment für den selektiven Happy-Case an sich ist recht einfach, hat aber hier und da seine Dinge, auf die man achten muss. Eine nicht ganz vollständige Liste, wenn man es später mit OpenSSH nutzen möchte:

* Private-Keys benötigen ein ``\n`` hinter dem ``-----END RSA PRIVATE KEY-----``
* Die Dateien müssen in der Regel nach ``~.ssh/`` abgelegt werden, welches dem User gehören und mit ``rwx------`` angelegt sein müssen.
* OpenSSH -- wenn keine weitere Konfiguration vorliegt -- nimmt alle Keys aus dem Verzeichnis und probiert diese durch. Das kann bei verschiedenen Systemen zu unoffensichtlichen "Zugriff verweigert"-Fehlern führen, wenn die Reihenfolge eine andere ist oder zu viele Keys im Verzeichnis vorliegen.
* Um sich zuverlässig und sicher mit der gewünschten Gegenstelle verbinden zu können, sollte eine ~.ssh/known_hosts gepflegt werden, welche sich die Fingerprints der Server merkt.
* OpenSSh-Keys sind in der Regel Plaintext-Dateien.
* Batou geht davon aus, dass es alleinig die Dinge managed, die im Deployment aufgeführt sind. D.h. bei unsauberer Benennung können Keys überschrieben werden und damit verloren gehen.

.. warning::
    Die Keys unbedingt und unabhängig davon, wie sie am Ende durch das Deployment auf den entfernten Host kommen, in den Secrets verwaren!

Manuelles deployen
------------------

Eine eigene Implementierung, die nicht alle der Punkte von oben aufgreift, könnte so aussehen:

.. code-block:: python

    from batou.component import Component, Attribute
    from batou.lib.file import File

    class SSHKeyPair(Component):

        priv_key = Attribute(str)
        public_key = Attribute(str)

        def configure(self):

            # Das Zielverzeichnis anlegen und sicherstellen, dass es
            # die richtigen Berechtigungen hat
            self += File('~/.ssh', ensure='directory', mode=0o700)

            # ED25519
            self += File(
                '~/.ssh/id_ed25519',
                content='{}\n'.format(self.priv_key),
                # OpenSSH benötigt auch für den Key strikte Berechtigungen
                mode=0o600,
                # Wir wollen die Ausgabe davon nicht in Gitlab-CI/Jenkins/*
                # sehen.
                sensitive_data=True)

            self += File('~/.ssh/id_ed25519.pub', content=self.public_key)


SSH-Keys mit batou_ext.ssh.SSHKeyPair
-------------------------------------

Einfacher ist es mit ``batou_ext.ssh.SSHKeyPair()``, welches einen großen Teil der regelmäßig verwendeten Muster im Deployment von SSH-Keys implementiert.

.. code-block:: python

    from batou_ext.ssh import SSHKeyPair


.. warning::
    Die Keys unbedingt in den Secrets verwaren!

.. code-block::

    [host:host01]
    components = sshkeypair

    [component:sshkeypair]
    id_ed25519 = -----BEGIN OPENSSH PRIVATE KEY-----
                 b3BlbnNzaC1rZXktdjEAAAAA …
                 -----END OPENSSH PRIVATE KEY-----
    id_ed25519_pub = ssh-ed25519 AAAAC3Nz …
    provide_itself = False


Ist ``provide_itself`` auf ``True`` gesetzt -- der Standardwert -- bietet sich die Komponente über ``sshkeypair`` anderen Komponenten zur Nutzung an. Wenn man also den SSH-Key direkt für einen git-Checkout benötigt, könnte der Stub der Komponente so aussehen

.. code-block:: python

    from batou.component import Component

    class MyCheckout(Component)

        def configure(self):
            self.ssh = self.require_one('sshkeypair', host=self.host)

Damit wird sichergestellt, dass das Deployment nicht startet, wenn die ``SSHKeyPair``-Komponente nicht auch dort zugewiesen ist, wo man ``MyCheckout`` braucht.

Die Komponente bietet noch ein paar zusätzliche Funktionen, wie ein unmanagen von alten Keys oder die Möglichkeit direkt die ``~.ssh/known_hosts`` zu aktualisieren -- Der letzte Schritt kann auch separat genutzt werden.


ScanHost um die known_hosts zu aktualsieren
-------------------------------------------

Möchte man unabhängig von der ``SSHKeyPair()``-Komponente die ``known_hosts`` aktualisieren, hilft ``batou_ext`` ebenfalls weiter:

.. code-block:: python

    from batou_ext.ssh import ScanHost
    from batou.component import Component

    class MyKnownHosts(Component):

        def configure(self):
            self += ScanHost('ssh.example.org', port=2222)
