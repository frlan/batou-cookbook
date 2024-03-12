Einen kompletten Host im Deployment (temporär) ignorieren
---------------------------------------------------------

Ab und an kommt es vor, dass man einen kompletten Host zwar definieren, aber
nicht anfassen will. Weil z. B. man seine Daten für Konfigurationen braucht
oder weil aus anderen Gründen gerade dort keine Änderung stattfinden darf. Man
kann einen Host entsprechend komplett im Environment 'deaktivieren'. 

.. code-block:: init
   [host:myhost]
   ignore = True
   components = … 

Das verwendete ignore ereldigt genau das. Der Host wird Teil der Berechnung,
aber nicht aktiv bedient. 

