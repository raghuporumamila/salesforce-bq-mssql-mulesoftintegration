# MuleSoft Integration: Salesforce to BigQuery & MS SQL

This project is a Mule 4 integration designed to sync Salesforce Account data into Google BigQuery (Analytics) and an on-premise MS SQL Server (Transactional). It is deployed to **Runtime Fabric (RTF)** via GitLab CI/CD.

## 🚀 Release Process

This project uses **GitLab Releases** integrated with the CI/CD pipeline.

### How to trigger a new Release:

1. Ensure your code is merged into `main`.
2. Create a new tag (following Semantic Versioning):
```bash
git tag -a v1.2.0 -m "Release notes: Added BigQuery streaming logic"
git push origin v1.2.0

```


3. The pipeline will automatically:
* Build the `.jar` artifact.
* Upload the artifact to the **GitLab Package Registry**.
* Publish the asset to **Anypoint Exchange**.
* Deploy the application to the **RTF Production Environment**.



---

## 🏗 Architecture Overview

* **Source:** Salesforce (Accounts/Contacts)
* **Transformation:** DataWeave 2.0 (Canonical Mapping)
* **Persistence:** * **BigQuery:** Streaming Insert for real-time analytics.
* **MS SQL:** JDBC Upsert for on-premise operational data.


* **Infrastructure:** Runtime Fabric (RTF) on Kubernetes.

---

## 🔐 Required CI/CD Variables

To run the pipeline, the following variables must be configured in **GitLab Settings > CI/CD > Variables**:

| Variable | Type | Description |
| --- | --- | --- |
| `MAVEN_SETTINGS` | **File** | The `settings.xml` containing Mule Enterprise credentials. |
| `ANYPOINT_USER` | Variable | Anypoint Platform username with Deployment permissions. |
| `ANYPOINT_PASS` | Variable | Anypoint Platform password (or Client ID/Secret). |
| `SF_TOKEN` | Masked | Salesforce Security Token for authentication. |
| `GCP_KEY_JSON` | **File** | Google Service Account JSON key for BigQuery access. |
| `MSSQL_PASS` | Masked | Password for the on-premise SQL Server. |

---

## 🛠 Local Development

To run this project locally, you must provide the properties via the command line or a local `dev-config.yaml`:

```bash
mvn clean mule:run -Dsf.user=... -Dsf.pass=... -Dmssql.host=localhost

```

---

## 📈 Monitoring & Logs

* **Deployment Status:** Viewable in GitLab under **Deploy > Releases**.
* **Application Logs:** Accessible via **Anypoint Monitoring** or `kubectl logs` within the RTF namespace.
* **Alerts:** Configured in Runtime Manager for 500-series errors or connection failures to MS SQL.

---
