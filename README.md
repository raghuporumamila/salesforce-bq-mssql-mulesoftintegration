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
In a professional production environment like **Runtime Fabric (RTF)**, a "rollback" is often just a "re-deployment" of a known stable version. Since GitLab keeps a history of every deployment, you don't necessarily need to write new code to fix a broken release.

### 1. The "Panic Button": GitLab UI Rollback

If a deployment fails or causes issues in Salesforce/SQL sync, GitLab has a built-in way to revert.

1. Navigate to **Operate > Environments**.
2. Click on your environment (e.g., `Production`).
3. You will see a list of successful past deployments.
4. Find the version that was working (e.g., `v1.1.0`) and click the **Rollback** button next to it.
* *Note:* This will trigger a new job that re-runs the `deploy` stage using the specific commit SHA of that old version.



---

### 2. Manual Rollback via Git Tags

If you prefer the command line or want to ensure a "clean" re-release, you can simply re-deploy an old tag.

```bash
# Verify which tags are available
git tag -l

# Re-trigger the pipeline for a previous stable tag
# (In GitLab, you can manually run a pipeline for a specific tag)

```

Go to **Build > Pipelines**, click **Run pipeline**, and select the stable tag (e.g., `v1.1.0`) from the dropdown.

---

### 3. MuleSoft Specific: Rollback via Anypoint Runtime Manager

If the GitLab pipeline itself is having issues, you can perform a rollback directly in the MuleSoft Control Plane.

1. Log in to **Anypoint Platform > Runtime Manager**.
2. Select your **RTF Application**.
3. Go to the **History** tab.
4. You will see a list of "Applied Configurations." Select a previous successful configuration and click **Apply**.
* **Why this is safe:** RTF stores the previous image and configuration state. This method is often faster than a full CI/CD run because the container image already exists in the local registry.



---

### 4. Critical Rollback Checklist for your Integration

When rolling back this specific Salesforce/BigQuery/SQL integration, check these three things immediately after the rollback:

* **Data Integrity (BigQuery):** Did the failed release create "junk" records? You may need to run a `DELETE` query in BigQuery for records processed during the failure window.
* **Connection Pools (MS SQL):** Ensure the rollback correctly re-established the connection to your on-premise SQL server.
* **Watermarking:** If you use "Object Store" for Salesforce watermarking (tracking the last record sync'd), you may need to manually reset the watermark if the failed version skipped records.

---

### Suggested README Section: Rollback Instructions

Add this to the `README.md` we created earlier:

```markdown
## 🆘 Emergency Rollback
In the event of a critical failure:
1. **Primary Method:** Go to GitLab `Operate > Environments > Production` and click **Rollback** on the last stable deployment.
2. **Secondary Method:** Go to Anypoint Runtime Manager, select the app, and use the **History** tab to re-apply the previous configuration.
3. **Database Check:** Verify that the `integration_audit` table in MS SQL hasn't been corrupted by the failed release.

```

