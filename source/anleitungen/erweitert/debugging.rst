Debugging
=========

Es kommt einmal vor, dass man bei einem Deployment auf Probleme stößt:

  * Eine Variable ist nicht korrekt gesetzt
  * Der Deployment-Code


Python Debugger
---------------

Ab der Version 2.3 von batou ist der `pdb` inkludiert, so dass man direkt in eine Debugger-Session starten kann. Durch das Setzen von `breakpoint()` kann man den Einsprungpunkt dabei wählen:

.. code-block:: python

        from batou.component import Component
        from batou.lib.file import File


        class MyComponent(Component)

            def configure(self):

                self += File(
                    'myfile.txt,
                    content='ein Dateiinhalt')

                breakpoint()

Führt man nun ein Deployment durch, kann man über z .B. telnet auf einem der Zielsystemen auf den pdb zugreifen. Die Asugabe könnte dann ungefähr so aussehen:

.. code-block::

    $ ./batou deploy dev

    batou/2.3b3.dev0 (cpython 3.9.9-final0, Darwin 20.6.0 x86_64)
    ====================================================== Preparing =======================================================
    main: Loading environment `testing`...
    main: Verifying repository ...
    You are using rsync. This is a non-verifying repository -- continuing on your own risk!
    main: Loading secrets ...
    ==================================================== Connecting ... ====================================================
    devhost00: Connecting via ssh (1/1)
    ================================================ Configuring model ... =================================================
    mux_client_request_session: session request failed: Session open refused by peer
    RemotePdb session open at 127.0.0.1:4444, waiting for connection ...

Nun kann man sich auf devhost00 einloggen sich mit dem Port 4444 verbinden

.. code-block:: shell

        telnet 127.0.0.1 4444

Und mit den typischen `pdb-Befehlen
<https://docs.python.org/3/library/pdb.html>`_ den Quellcode debuggen.

