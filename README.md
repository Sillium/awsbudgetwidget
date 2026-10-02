# awsbudget

Dies ist eine AWS-Lambda-Funktion, die die essentiellen Daten von AWS Budgets als JSON liefert. Sie kann bspw. als Backend für iOS Scriptable Widgets genutzt werden.

![Diagramm](diagram.drawio.svg "Funktionsweise")

## Installation

Die `serverless.yml` zum Deployment per Serverless Framework (`frameworkVersion: '2'`) ist inkludiert. Die Lambda läuft mit Python 3.8 in `eu-central-1`.

Voraussetzungen: Node.js/npm, Serverless Framework, Python 3 (`python3`).

```
npm install
pip install -r requirements.txt
serverless deploy --stage <dev|prod> --param="accountId=<aws_account_id>" --param="certificateArn=<acm_certificate_arn>"
```

- `--stage` ist Pflicht und muss `dev` oder `prod` sein. Er bestimmt die Domain (`dev.awsbudgetwidget.sillium.xyz` bzw. `awsbudgetwidget.sillium.xyz`) und das Log-Level (`DEBUG` bzw. `INFO`, als Umgebungsvariable `LOG_LEVEL`).
- Parameter `accountId`: AWS-Account für den Deployment-Bucket (`serverless-deployments-<accountId>`) und den CloudFront-Logging-Bucket (`cloudfront-logs-<accountId>`).
- Parameter `certificateArn`: ARN des Zertifikats für die CloudFront-Distribution.
- Plugins: `serverless-python-requirements` und `serverless-api-cloudfront` (der API Gateway wird per CloudFront ausgeliefert).

## Aufruf

Die Lambda kann per API-Gateway-Endpunkt aufgerufen werden, also bpsw. `https://<api_gateway_id>.execute-api.eu-central-1.amazonaws.com/prod/budget/<aws_account_id>/<budget_name>`.

Die Autorisierung erfolgt über einen IAM User und eine IAM Role. Die IAM Role muss die Berechtigung haben, das AWS Budget im übergebenen AWS Account zu lesen. Der IAM User muss die Berechtigung haben, diese IAM Role zu assumen. Folgende 3 Header müssen beim Funktionsaufruf mitgegeben werden:
    - aws_role_name
    - aws_access_key_id
    - aws_secret_access_key
    
## Caching

Das API Gateway cached Aufrufe für 1h. Cache-Keys sind alle übergebenen Parameter. Das soll verhindern, dass ein unberechtigter Aufruf (ohne übergebenen IAM User) ein gecachetes Ergebnis erhält.

Caching ist auf jeden Fall sinnvoll, da die AWS Cost Explorer API Kosten von USD 0.01 pro Aufruf verursacht.

## Scriptable-Widget

Unter `scriptable/OBI-AWS-Budget.js` liegt ein Beispiel-Skript für iOS Scriptable. Im Skript müssen `aws_access_key_id`, `aws_secret_access_key` und `budgetApiEndpoint` (Platzhalter) angepasst werden. Das Widget wird über einen JSON-Widget-Parameter konfiguriert, z.B. `{ "accountId": "<account_id>", "budgetName": "<budget_name>", "title1": "<project>", "title2": "<stage>" }`.

## Projektstruktur

- `functions/getBudget.py` – Lambda-Handler (assumed die IAM Role, liest das Budget und den Account-Alias)
- `serverless.yml` – Deployment-Konfiguration
- `scriptable/` – iOS-Scriptable-Widget
- `diagram.drawio.svg` – Diagramm der Funktionsweise
- `package.json`, `requirements.txt` – Abhängigkeiten (Serverless-Plugins bzw. Python)
- `.gitpod.yml` – Gitpod-Setup (`npm install && pip install -r requirements.txt`)
