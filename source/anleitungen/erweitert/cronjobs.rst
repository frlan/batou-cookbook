CronJob-Handling
================

Löschen von CronJobs
--------------------

Batou geht davon aus, dass es die CronTab für den Service-Nutzer managed und generiert bei der Verwendung der CronTab-Komponentne eine vollständige Tabelle. Entfernt man später aber CronJobs vone iner Ziel-Umgebung vollständig, wird die CronTab nicht neu geschreiben und die vielleicht noch vorhandenen Einträge bleiben erhalten. Das kann zu unerwünschten Ergebnissen führen.

Das Problem kann man durch die Verwendung von batou.lib.cron.PurgeCronTab lösen:

.. code-block:: python

        from batou.lib.cron import PurgeCronTab

Fügt man nun die Komponenten ``purgecrontab`` hinzu, wird die Tabelle beim nächsten Deployment gelöscht.
