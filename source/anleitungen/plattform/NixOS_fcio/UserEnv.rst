NixOS UserEnv mit batou_ext
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Um Bibliotheken oder Anwendungen in einer zielumgebng zu innen, kann man auf NixOS ein UserEnv in der Nutzer installieren. Das, was man mit `nix-env -if`oder mit `nix-env -iA` auf der Kommdoziele auf dem Zielsystem macht, kann man auch über batou im Deployment definiert deployen.


Einfaches UserEnv mit gepinnten NixOS-Channel
=============================================

Mit Hilfe bon `batou_ext.nix.UserEnv` kann man eine einfache Umgebung in das Home und damit das Profil des Nutzer installieren:

.. code-block:: python

    from batou_ext.nix import UserEnv
    from batou.component import Component


    class MyComponente(Component):

        def configure(self):
            self += batou_ext.nix.UserEnv(
                "my_node_env",
                packages=[
                    "mysql57",
                    "nodejs-12_x",
                    "zip",
                ],
                channel=("https://releases.nixos.org/nixos/19.03/nixos-19.03.173691.34c7eb7545d/nixexprs.tar.xz"))


`packages` übernimmt dabei eine Liste von Programmen/Biblitohken, die man im Profil benötigt, `channel` die URL zu einem komkreten NixSO-Channel, z. B. von https://nixos.org/channels/.
