Referenzierung von Komponenten
==============================

Batou kann Abhängigkeiten zwischen Komponenten abbilden. Das kann sinnvoll sein, wenn eine Komponenten ohne eine andere nicht funktioniert -- z. B. ist ein ``git clone`` eines externen Quellcode-Repositories ohne einen SSH-Key nicht möglich -- oder wenn auf Daten einer anderen Komponente zugegriffen werden soll. In vielen Fällen ist eine genaue Trennlinie nicht zu ziehen und das ist zum Glück auch nicht notwendig

Provide und Require
-------------------


.. code-block:: python

	from batou.component import Component

	class MyComponentProvide(Component):

		def configure(self):
			self.provide('mycomponent1', self)

	class MyComponentRequire(Component):

		def configure(self):
			self.comp1 = self.require_one('mycomponent1')


Umgekehrte Beziehung
--------------------

Sanfte Beziehungen
------------------
