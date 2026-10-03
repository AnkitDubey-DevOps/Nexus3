### Advanced Usage

**51. How do you configure Nexus for high availability?**

HA is a **Pro** feature. You run **multiple Nexus nodes** behind a **load balancer**, with a shared **external database (PostgreSQL)** and a **shared blob store** (such as S3). If one node fails, the others keep working.

**52. What are the benefits of Nexus Pro?**

High availability, staging and promotion features, replication, **SAML single sign-on**, security scanning with Nexus Firewall/IQ, and **professional support** from Sonatype.

**53. How do you manage npm packages?**

Create an **npm hosted** repository (your own packages), an **npm proxy** (to npmjs.org), and an **npm group** combining them. Point npm to the group URL with `npm config set registry <url>`, and publish with `npm publish`.

**54. What is the purpose of the REST API?**

It lets scripts and tools **control Nexus automatically**: create repositories, upload and search components, manage users and roles, run tasks.

**55. How do you use the Nexus CLI?**

⚠️ **Update:** Nexus 3 has **no official CLI**. Some community tools exist, but the normal way is **`curl` with the REST API**, or build tools like Maven and npm. Say: "We automate with the REST API."

---

### Common Commands and Configurations

**56. How do you configure Maven to use Nexus?**

In `settings.xml`: add a **mirror** pointing to the Nexus group URL, and add your login under `<servers>`. In `pom.xml`: add `<distributionManagement>` for deploying. The server IDs must match.

**57. How do you push a Docker image to Nexus?**

`docker login <nexus-host:port>`, then `docker tag myapp <nexus-host:port>/myapp:1.0`, then `docker push <nexus-host:port>/myapp:1.0`.

**58. How do you pull a Docker image from Nexus?**

`docker pull <nexus-host:port>/myapp:1.0`.

**59. How do you configure a cleanup policy?**

Go to **Repository → Cleanup Policies**, create a policy (for example, "not downloaded in 60 days"), and **assign it to a repository**. Then run **Compact blob store** to free the disk space.

**60. How do you set up a proxy repository for Maven Central?**

Create a **maven2 (proxy)** repository, set the remote URL to `https://repo1.maven.org/maven2/`, choose the **Release** version policy, and keep the default cache settings. Add it to your `maven-public` group.

---

### Detailed Questions and Answers

**61. How do you use a custom port?**

Edit `nexus.properties` in the data folder's `etc` directory (`sonatype-work/nexus3/etc/` or `nexus-data/etc/`), change `application-port=8081` to your port, and **restart** Nexus.

**62. How do you create a Docker group repository?**

**Repositories → Create repository → docker (group)**, give it its own HTTP/HTTPS port, add the Docker hosted and proxy repositories as members, and save. Enable the **Docker Bearer Token realm**.

**63. How do you set up PyPI?**

Create **PyPI hosted** (internal packages), **PyPI proxy** (to pypi.org), and a **PyPI group**. Install with `pip install <pkg> --index-url <nexus-group-url>/simple`, and upload with `twine`.

**64. How do you handle large uploads?**

⚠️ **Update:** There isn't usually a "max upload size" setting in Nexus itself. Problems normally come from the **reverse proxy** (for example, Nginx `client_max_body_size`), **timeouts**, low **disk space**, or low **memory**. Check those first.

**65. What is the role of `nexus-context.xml`?**

⚠️ **Update:** That file belonged to **Nexus 2**. In Nexus 3, the main settings are in `nexus.properties`, `nexus.vmoptions`, and the Jetty config files under `etc/`. Most configuration is done in the web UI and stored in the database.

**66. How do you configure email notifications?**

Go to **Administration → System → Email Server**, enter the SMTP details, and test. Email is used for things like **password reset** and **task failure alerts**. (Pro features can use it for more.)

**67. What is the Nexus Audit Plugin?**

⚠️ **Update:** In Nexus 3 it is a **built-in Audit capability**, not a separate plugin. It records who did what and when (logins, changes, uploads, deletions). Turn it on in the capabilities settings.

**68. How do you configure repository health checks?**

⚠️ **Update:** The old **Repository Health Check (RHC)** feature depends on the version and edition. A safe answer: "I monitor repository health with the **status page, proxy repository status, metrics, and logs**, plus security scanning with Nexus IQ."

**69. How do you manage SSL certificates?**

Two ways: put a **reverse proxy** (Nginx/Apache) in front and install the certificate there, or put the certificate in a **Java keystore** and enable HTTPS in Nexus. Set a reminder to **renew before expiry**.

**70. How do you set up Nexus for multiple organizations?**

Create **separate repositories per organization**, use **roles and content selectors** so each group sees only its own content, and use separate blob stores if you need to track storage. For strict isolation, use separate Nexus instances.

---

### In-depth Questions

**71. What performance tuning options exist?**

⚠️ **Update:** Skip "index optimization" (old). Say: increase **JVM heap** and direct memory, use **SSD disks**, raise the **file descriptor limit**, run **cleanup and compact** tasks, split blob stores, and run heavy tasks off-peak.

**72. How do you set up disaster recovery?**

Take **regular backups** (database and blob stores), store a copy **off-site**, keep a **documented and tested restore plan**, and optionally have a standby site. Remember: **HA is not DR**.

**73. How do you migrate artifacts to another Nexus?**

⚠️ **Update:** There is no simple "export repositories" button. Common ways: **restore the database and blob store backup** on the new server (same version), **copy artifacts using the REST API or scripts**, or use Sonatype's **migration tools** for Nexus 2 → 3. Verify with checksums afterward.

**74. How do you integrate other CI tools?**

Use the **REST API**, or build-tool commands inside the pipeline (`mvn deploy`, `npm publish`, `docker push`). This works with **GitLab CI, GitHub Actions, Azure DevOps, Bamboo, CircleCI**, and others.

**75. How do you configure Nexus for GDPR?**

GDPR is about **personal data**. In Nexus that means user accounts, emails, and IP addresses in logs. Use **access control**, **log retention limits**, remove users when they leave, and keep **audit logs**. Check the exact rules with your compliance team.

**76. Benefits of Nexus over other repository managers?**

Free edition available, **many formats in one tool**, strong integration with **Sonatype security scanning**, and wide CI/CD support. Be fair: alternatives like **JFrog Artifactory** are also strong, and the best choice depends on needs and budget.

**77. How do you handle artifact versioning?**

Use **semantic versioning** (`MAJOR.MINOR.PATCH`), keep **snapshots and releases in separate repositories**, and make **releases immutable** (no redeploy).

**78. What is the impact of cleanup policies on build performance?**

They keep repositories small, which saves **disk space, backup time, and search load**. Be careful: deleting something a project still needs will **break builds**, so never clean up releases still in use.

**79. How do you configure an external database?**

⚠️ **Update:** Older Nexus used embedded **OrientDB**. **Newer versions** use **H2** by default and support **PostgreSQL** (needed for HA). Check the docs for your version before answering.

**80. How do you monitor usage and activity?**

Use the **status page**, **logs**, **audit log**, and a **metrics endpoint** (Prometheus/Grafana), plus alerts for disk, memory, and response time.

---

### Advanced Configuration

**81. What is the purpose of Nexus Firewall?**

It checks components coming from proxy repositories and **blocks or quarantines** the ones with known vulnerabilities or bad licenses, **before** they enter your network. (Separate licensed Sonatype product.)

**82. How do you configure Nexus behind a proxy server?**

Go to **Administration → System → HTTP**, enter the outbound proxy host, port, and login. Nexus then uses it to reach remote repositories like Maven Central.

**83. What are Blob Stores and how do you manage them?**

They hold the **actual artifact files**. Create them under **Repository → Blob Stores**, use several for different workloads, set **soft quota alerts**, **compact** regularly, and **back them up**.

**84. How do you configure SSO?**

⚠️ **Update:** **LDAP is central login, not true SSO.** True SSO uses **SAML (Pro)**, or a reverse proxy passing a trusted user header (the Remote User Token realm). Check which your version supports.

**85. Best practices for large-scale deployments?**

**HA** (Pro), external PostgreSQL, shared or cloud blob stores, **cleanup policies**, monitoring and alerts, tested backups, **infrastructure-as-code**, and regular upgrades.

**86. How do you support many languages and build tools?**

Create a repository set (hosted, proxy, group) **per format** (Maven, npm, PyPI, Docker), point each build tool at its group URL, and give each team the right roles.

**87. How do you handle promotion in a multi-stage pipeline?**

Publish to a **staging/dev repository**, run tests and scans, and **promote the same artifact** to the next repository on success (REST API, scripts, or Pro staging). Never rebuild.

**88. How do you set up automated cleanup?**

Create **cleanup policies**, assign them to repositories, make sure the **cleanup task** is scheduled, and also schedule **Compact blob store** so space is really freed.

**89. How do you integrate Nexus with version control?**

⚠️ **Update:** Nexus doesn't connect to Git directly. The link goes through **CI**: a Git commit triggers a pipeline, which builds and **pushes the artifact to Nexus**. Include the commit ID in the build info.

**90. How do you support distributed teams?**

Use **regional Nexus instances** (proxying the main one, or Pro replication), **RBAC** for each team, and cloud storage for blob stores. (HA clustering is for failover, not for geography.)

---

### Troubleshooting and Maintenance

**91. How do you fix repository synchronization issues?**

For proxy repositories: check the **remote URL**, network and firewall, whether the repository is **auto-blocked**, then **invalidate the cache** and rebuild metadata if needed. Check the logs.

**92. What if Nexus has high CPU usage?**

Check which **scheduled task** is running (cleanup, compact, rebuild), look for **garbage collection** or memory pressure, review heavy requests in logs, and then tune the heap, clean up, or add CPU.

**93. How do you handle failed uploads?**

Check these, in order: **permission** (deploy privilege), **write policy** (releases usually block redeploy), **version policy mismatch** (snapshot sent to a release repository), **credentials/server ID**, **disk space**, and **proxy limits or timeouts**. Then read the logs.

**94. How do you upgrade Nexus?**

**Back up first**. Stop Nexus, install the new version, and point it at the **same data directory**. Start and test. For big version jumps, follow the official upgrade path, as the database may need migration.

**95. How do you troubleshoot startup issues?**

Read `nexus.log`, check the **Java version**, **file permissions** and the user running Nexus, **disk space**, and whether the **port is already in use**.

**96. How do you monitor and manage performance?**

Track **CPU, heap, disk, and response times** using the status page, metrics, and logs. Schedule maintenance tasks and alert on trends.

**97. How do you handle large-scale storage?**

Use **multiple blob stores** (or cloud storage like S3), set **quotas and alerts**, apply **cleanup policies**, compact regularly, and plan disk growth.

**98. How do you ensure high availability?**

Use **HA nodes with a load balancer**, PostgreSQL, and a shared blob store (Pro), plus **health checks** and tested failover. Keep backups too, since **HA is not a backup**.

**99. Best practices for securing repositories?**

Change default passwords, use **HTTPS**, **least-privilege RBAC**, disable anonymous access if not needed, use LDAP/SSO, run Nexus as a **non-root user**, keep it **updated**, scan components, and watch the **audit logs**.

**100. How do you manage multiple Nexus instances?**

⚠️ **Update:** Nexus has no single central console. Use **infrastructure-as-code** (Terraform/Ansible with the REST API) for **identical configuration**, shared standards, replication or proxying between instances, and common **monitoring and backup** processes.
