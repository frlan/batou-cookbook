Komponenten zum Informationsaustausch
=====================================

Eine einfache Methode zum Teilen von Informationen von zwischen verschiedenen Komponenten/Anwendungskomponenten sind gluing-Komponeten, deren einzigstes Ziel es im Grunde ist, Informationen für andere Komponnetenten zur Verfügung zu stellen. Entsprechend einfach sie zu erstellen und benutzen.

Sie müssen im Grunde zwei Dinge besitzen:

* Die Informationen, die man austauschen möchte
* Ein ``self.provide()``, so dass man sie in andere Komponenten einbinden kann

.. code-block:: python

    from batou.component import Component
    from batou.component import Attribute

    class MySharingComponent(Component):

        pfad1 = Attribute(str, '/ich/bin/ein/Pfad')
        wahrheit = Attribute('literal', False)
        zahl1 = Attribute(int, 23)

        def configure(self:
            self.provide('mysharingcomponent', self)


Für man sie zum Environment hinzu:

.. code-block::

    [host:host01]
    components = mysharingcomponent


kann man diese Werte einfach in anderen Komponenten nutzen

.. code-block:: python

    from batou.component import Component
    from batou.lib.file import File

    class MyConsumerComponent(Component):

        def configure(self):

            self.data = self.require_one('mysharingcomponent)

            if self.data.zahl1 == 23:

                self += File(self.data.pfad1, content=self.data.wahrheit)

Besondere Fälle davon sind:

* `batou_ext.nfs.NFS`_
* `Settings`