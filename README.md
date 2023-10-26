# thb-build-actions
Contains reusable GitHub Actions

Um eine Maven Bibliothek zu benutzen, müssen in den `build-java-input-command` Workflow folgende Input-Secrets eingegeben werden:
- repo-access-username
  description: A username wich owns a repository read access token

- repo-access-token
  description: The personal access token of the passed repo-access-username

Diese Werte betiteln einen Benutzernamen und ein Token, welches read-access auf das gewünschte Repositories mit der Maven Bibliothek besitzt.
Werden diese BEIDEN Werte aus dem Repository Secrets vom aufrufenden Workflow reingegeben, so wird diese Action eine settings.xml anlegen, die die Lese-Berechtigung für die GitHub Packages beinhaltet. Wird nur einer oder kein Wert übergeben, so wird auch keine settings.xml angelegt.

Ist in der pom.xml eines Projekts, welches mithilfe des `build-java-input-command` Workflows gebaut wird, nun ein <repositories> Tag enthalten, welches auf thb-it-oldenburg/[repository-name] verweist, so wird die Maven Bibliothek aus diesem Repository geladen.

Zusammenfassung der Tätigkeiten, um eine Maven Bibliothek zu nutzen:
- Das Repository, in dem die Bibliothek liegt, in den <repositories> Tag der pom.xml des zu bauenden Projekts eintragen
- Einen Usernamen und den zugehörigen Personal-Access-Token in die Repository Secrets des zu bauenden Projekts eintragen
- Diesem User mindestens Lesezugriff auf das Repository geben, in dem die Bibliothek liegt.
