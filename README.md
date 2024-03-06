# thb-build-actions
Contains reusable GitHub Actions

# Maven Bibliotheken
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


# Dockerfile build secrets
Der Workflow `build-container-push-to-registry` stellt das Github Token in dem Dockerfile über `docker build --secret id=github_token,src=./github_token.txt` bereit.

Dadurch kann das Dockerfile zum Beispiel auf die GitHub Registry zugreifen. Wenn im Dockerfile der Befehl:

`RUN --mount=type=secret,id=github_token`

steht, liegt das Token anschließend im Dockerfile /run/secrets/github_token. Da diese secret Dateien nicht persistiert werden, sondern in-memory nur kurzzeitig gespeichert werden, wird das Token nach dem Build Prozess nicht in dem Image landen!

## Willkürliche Secret Datei
Es ist es möglich eine Datei als Secret zu übergeben. Dafür müssen die Keys `secret.file-name` und `secret.file-content` ausgefüllt werden.
Diese Datei ist zur freien Verwendung gedacht und steht im Docker build Prozess zur Verfügung. Du kannst sie also im Dockerfile verwenden.

Beachte, dass der Inhalt von `secret.file-content` base64 verschlüsselt sein muss.
Dafür kannst du in cygwin `echo "meinText" | base64` ausführen.

# Flutter build and deploy to playstore
## Secrets
### SERVICE_ACCOUNT_KEY_FILE_CONTENT_BASE64
Enthält die Zugriffsrechte für ein Dienstkonto der Google Cloud, welches die Berechtigung haben sollte, auf die gewünschte App im Playstore zuzugreifen
Diese Datei hat folgendes Format:
```json
{
  "type": "service_account",
  "project_id": "thb-timetracker",
  "private_key_id": "Eine Schlüssel ID in alphanumerischen Zeichen",
  "private_key": "Ein voll geheimer Schlüssel",
  "client_email": "email-des-dienstkontos@projekt.iam.gserviceaccount.jcom",
  "client_id": "Eine Client ID in Ziffern",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.Eine längere URL.com"
  "universe_domain": "googleapis.com"
}
```
Da der private_key und die Datei selbst newlines enthält, muss der gesamte inhalt base64 codiert werden, da sonst die bash commands nicht funktionieren.

### KEYSTORE_INFORMATION_FILE_BASE64
Enthält den Pfad und das Passwort für den keystore, welcher den privaten upload Key enthält.
Die Datei hat folgendes Format:
```
storePassword=EinKlartextPasswortFürDenStore
keyPassword=EinKlartextPasswortFürDenZuVerwendendenKey
keyAlias=AliasNameDesKeys z.B. upload
storeFile=D:\\Pfad\\zum\\keystore.jks
```
Auch diese Datei muss base64 codiert werden.

### KEYSTORE_SIGNING_KEY_BASE64
Enthält den keystore, also eine .jks Datei, welche den privaten upload Key für die Appbundles enthält. Ohne sie wird die App nicht signiert und vom Playstore abgelehnt.
Die genaue Form ist unbekannt, sie muss aber in einem Secret base64 codiert vorliegen, damit der workflow die Datei korrekt im Runner anlegen kann.
