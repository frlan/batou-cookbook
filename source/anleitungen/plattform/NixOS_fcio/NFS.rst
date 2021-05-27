NFS
###

batou_ext.nfs.NFS
-----------------

.. batou_ext.nfs.NFS:

Die Komponente ist eine typische `glue component`. Sie ist dafür da, dass man die Basis-Pfade eines NFS-Setups an einer zentralen Stelle im Deployment zu pflegen, um später drauf in allen Komponenten zugreifen zu können.

Um sie zu nutzen, muss man im einfachsten Fall sie nur an einer zentralen importieren und im Environment einschalten:

.. code-block:: python

    from batou_ext.nfs import NFS


.. code-block::

    [host:host01]
    components = nfs

Die Komponente greift in der Stadard-Einstellung auf die Pfade bei FCIO zurück, diese können aber einfach an die lokalen Gegebenheiten angepasst werden

.. code-block::

    [host:host01]
    components = nfs

    [component:nfs]
    basepath = /pfad/zum/mountpoint/auf/den/clients
    serverpath = /pfad/zum/servershare/auf/dem/nfs/server

Jetzt kann man auf diese Variablen im Deployment einfach zugreifen und muss sich keine Gedanken mehr über hartkodierte Pfade machen, wenn sich die Konfiguration an anderer Stelle ändert.

.. code-block:: python

	from batou_ext.nfs import NFS
	from batou.component import Component
	from batou.lib.file import Directory

	class myComponent(Component):

	    def configure(self):

	        self.nfs = self.require_one('nfs')
	        # Wir stellen sicher, ein Verzeichnis auf dem NFS-Mount existiert
	        self += Directory(f'{self.nfs.basepath}MeinVerzeichnis')
