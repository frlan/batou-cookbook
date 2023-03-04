Python VirtualEnvs
==================

Batou bietet verschiedene Möglichkeiten, um Pakete in ein VirtualEnv zu insallieren. Allen Möglichkeien ist gemein, dass das venv alle Komponenten in ``work/.virtualenv`` installiert wird.

batou.lib.python.VirtualEnv
----------------------------

Manchmal braucht man nur ein sehr minimalistisches Python VirtualEnv, in
welchens man später Pakete mit pip installieren möchte. 


.. code-block:: python

    self += VirtualEnv("3.10")
    self.venv = self._

Erstellt ein VirtualEnv mit einem Python 3.10 und weißt es der Variable venv
zu, so dass es später weiterverwendet werden kann. Weiterverwendet in dem man
zum Beispiel dort Pakete rein installiert:

.. code-block:: python

    self += VirtualEnv("3.10")
    self.venv = self._
    self += self.venv.Package(
        "django",
        version="4.1.7")


Bei größeren Projekten möchte man aber vielleicht einen direkteren Weg
einschlagen:

batou_ext.python.VirtualEnvRequirements
---------------------------------------

In vielen Projekten hat man eine ``requirements.txt``, um benötigte Pakete zu installieren -- hier einmal am Beispiel eines Checkouts via ```batou_ext.git.GitCheckout``

.. code-block:: python

        self += VirtualEnvRequirements(
            version='3.8',
            requirements_path=f'{self.checkout.prepared_path}/requirements.txt')
        self.venv = self._


