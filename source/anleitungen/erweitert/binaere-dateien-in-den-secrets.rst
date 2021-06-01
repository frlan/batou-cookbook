Binäre Dateien in den Secrets
=============================

.. Binaere_dateien_in_den_secrets:

.. code-block:: python

    import base64

    with open('dateiname', "rb") as f:
        encodedf = base64.b64encode(f.read())
        print(encodedf.decode())

Damit kann man die Daten über ``./batou secrets edit <environment> <name>`` in eine dedizierte Datei schreiben oder einfach als Variable in den Secrets hinterlegen.

Im Deployment an sich muss man natürlich die Aten wieder zurückkonvertieren. Für das Beispiel wurden die Daten in ``binardaten`` abgelegt.


.. code-block:: python

    from batou.lib.file import BinaryFile
    from batou.component import Component

    import base64


    class MyComponent(Component):

        def configure(self):
            self += BinaryFile(
                'meineBinärdatei',
                content=base64.decodebytes(
                    self.environment.secret_files['binardaten'].encode()))
