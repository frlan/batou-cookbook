Befehle ausführen
=================

Immer wieder benötigt man Möglichkeiten, um Befehle auf dem Zielsystem aufzurufen. Sei es un eine Datenbankmigration anzustoßen, einen Buildprozess zu starten oder schlicht einige Dateioperationen durchzuführen.

Einfache Befehle aufrufen
-------------------------

.. warning::
	``self.cmd()`` sollte nur in ``verify()`` oder ``update()`` aufgerufen werden, da Anweisungen aus der ``configure()``-Mothode auf allen Hosts ausgeführt wird.

.. hint::
	``self.cmd()`` ist gut geeignet zum Ausführen von einfachen Befehlen, die keine großen Anforderungen an die Umgebung setzen.


Komplexere Skripte mit batou_ext.run.Run()
------------------------------------------

t.b.p.s.
