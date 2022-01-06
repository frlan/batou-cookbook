Requirements pinning für batou
=============================

Genereller requiremets.txt-Ansatz
---------------------------------
Batou ist eine Python-Anwendung. Im Hintergrund installiert es die in der requirements.txt definierten Abhängigkeiten in ein virtualenv und führt sich dann aus.

Batou <2.0
----------

In den Versionen vor dem 2.0-Release, war die Version von batou im Script hart kodiert, so dass eine requirements.txt nicht unbedingt notwendig wurde, wenn man keine zusätzlichen Bibliotheken wie `boto` oder `batou_ext` benötigte.

Ein Versionsupdate von batou konnte über eine Buildin-Funktion erledigt werden.

Batou 2.x
---------

Mit der 2.0-Release wurde `batou` auf die Verwendung von `appenv` umgestellt. `appenv` an sich, ist eine Möglichkeit, um andere Pythonporgramme einfach in desem Kontext auszuführen. Entsprechend werden die Progammversionen über die `requirements.txt`definiert. Die Installation von batou via `appenv` legt diese Datei schon richtig mit einem passenden Eintrag an. Dieser könnte so aussehen

.. code-block:: sh

        batou==2.2.4


Neu ab Version 2 ist die Möglichkeit, alle Abhängigkeiten zu pinnen. Dies hat zwei Vorteile

* Genaue Definition der verwendeten Python-Pakete und damit bessere Reproduzierbarkeit
* Schnellere Ausfürhungszeiten auf lokal, wie auch auf den Zielservern, da so die Umgebung von appenb bzw. batou nicht mit jedem deploymentlauf neu initialisiert werden muss.

Versionspinning mit 2.2.x
*************************

Durch den Aufruf von

.. code-block:: sh

        ./batou appenv-update-lockfile

erzeugt `batou` eine `requirements.lock`, in welche alle direkten und indirekten Abhängigkeiten zum Ausführen des Deployment an sich aufgenommen werden.


Versionspinning mit 2.2.x
*************************

Mit Version 2.3 wurde die appenv-Integration von batou überarbeitet. Dadurch änderte sich auch die Art und Weise, wie bestimmte Befehle aufgerufen werden. Das aus früheren Versionen bekannte `update-lockfile` ist eigentlich eine Funktionalität von `appenv`, entsprechend wird es seit der Version 2.3.x über

.. code-block:: sh

        ./appenv update-lockfile

aufgerufen.

Für die Berechnung der Abhängigkeiten wird dabei das Systempython genutzt. Möchte man das Deployment auf Servern laufen lassen, die andere Versionen von Python benutzen, kann es passieren, dass Abängigkeiten falsch definiert sind oder ausgelassen wurden. Dies kann durch das setzen einer entsprechenden Markierung in der `requirements.txt` angepast werden. Ein Beispiel für die Unterstütung von Python 3.6, 3.8 und 3.9 auf den Zielsystemen könnte entsprechend so aussehen:

.. code-block::

        # appenv-python-preference: 3.8,3.9,3.6

Dadurch werden die vom Python 3.6 explizit benötigten Pakete wie importlib-metadata oder typing_extensions in die so entstehende `requirements.lock` mit aufgenommen.

