JSON-Dateien mit Batou
======================
Seit: 2.1

Batou unterstützt das direkte Schreiben von JSON-Dateien -- kein json.dumps oder aufwendiges Templaten von JSON:


.. code-block:: python

    import batou.lib.file

    self += batou.lib.file.JSONContent()



Wie gewohnt, übernimmt JSONContent den Namen der Datei, welche geschrieben werden soll

.. code:: python

    import batou.lib.file

    self += batou.lib.file.JSONContent('myJSON.json', )


Der Inhalt kann über verschiedene Wege definiert werden:

Neue Datei anlegen
------------------

Zum Anlegen eines neuen JSON-Files muss man der Komponente ein Python-Dictionary mit der gewünschten Struktur und Daten übergeben:

.. code:: python

    import batou.lib.file

    mydata={
        'version': '1.23',
        'debug': True,
        'size': 42
    }

    self += batou.lib.file.JSONContent(
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

Manchmal muss man auch nur eine existierende Datei mit ein paarWerten patchen.

upstream.json

.. code-block:: json

    {
        "height": "1,23",
        "size": 23
    }


.. code-block:: python

    import batou.lib.file

    mydata={
        'version': '1.23',
        'debug': True,
        'size': 42}

    self += batou.lib.file.JSONContent(
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


Aber Achtung: Im Hintergrund wird dabei ein Dictmerge ausgeführt -- das füh rt dazu, dass Liste erweitert werden, nicht ersetzt.
