JSON-Dateien mit Batou
======================

Batou unterstützt das direkte Schreiben von JSON-Dateien -- kein json.dumps oder aufwendiges Templaten von JSON notwendig:


.. code-block:: python

    from batou.component import Component
    from batou.lib.file import JSONContent

    class MyComponent(Component):
        self += JSONContent()


Wie gewohnt, übernimmt JSONContent den Namen der Datei, welche geschrieben werden soll, sowie ein Python-Dictionary mit den gewünschten Daten:

.. code:: python

    self += JSONContent('myJSON.json', data=dict())


Der Inhalt kann über verschiedene Wege definiert werden:

Neue Datei anlegen
------------------

Zum Anlegen eines neuen JSON-Files muss man der Komponente ein Python-Dictionary mit der gewünschten Struktur und Daten übergeben:

.. code:: python

    from batou.lib.file import JSONContent
    from batou.component import Component


    class MyComponent(Component):

        def configure(self):

            mydata={
                'version': '1.23',
                'debug': True,
                'size': 42
            }

            self += JSONContent(
                'myData.json',
                data=mydata)


Batou legt nun eine Datei `myData.json` an, dessen Inhalt

.. code-block:: json

    {
        "version": "1.23",
        "debug": true,
        "size": 42
    }

ist.


Eine existierendes JSON-File patchen
------------------------------------

Manchmal muss man auch nur eine existierende Datei -- zum Beispiel aus einem Quellcode-Repository --  mit ein paar Werten patchen.

Nehmen wir diese ``upstream.json`` an:

.. code-block:: json

    {
        "height": "1,23",
        "size": 23
    }


.. code-block:: python

    from batou.lib.file import JSONContent
    from batou.component import Component


    class MyComponent(Component):

        def configure(self):

            mydata={
                'version': '1.23',
                'debug': True,
                'size': 42}

        self += JSONContent(
             'myData.json',
             data='upstream.json',
             override=mydata)


Daraus wird entstehen

.. code-block:: json

    {
        "height": "1,23",
        "version": "1.23",
        "debug": true,
        "size": 42
    }


.. warning::
    Im Hintergrund wird dabei ein Dictmerge ausgeführt -- das führt dazu, dass eine vielleicht vorhandene Liste erweitert werden, nicht ersetzt.
