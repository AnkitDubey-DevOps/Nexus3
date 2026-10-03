### 1. Centralized Artifact Management

1. Nexus is a single repository manager that stores all build artifacts (Maven `.jar`, npm packages, Docker images) in one place.
2. It acts as the **single source of truth** for developers and CI/CD pipelines, ensuring consistent dependency versions.
3. It eliminates "works on my machine" issues caused by version mismatches.
4. Supports multiple formats (Maven, npm, Docker, PyPI, NuGet, etc.) in one tool.
5. Provides access control, auditing, and security scanning of artifacts.

### 2. Proxy Remote Repositories

1. A proxy repository sits between your team and a remote repository like Maven Central or npmjs.org.
2. On first request, Nexus downloads the artifact and **caches** it locally; later requests are served from the cache.
3. Speeds up dependency resolution and builds.
4. Reduces external bandwidth usage and download costs.
5. Provides resilience: builds continue even if the remote repository is down.

### 3. Hosted Repositories for Internal Projects

1. A hosted repository stores artifacts your organization creates (internal libraries, in-house tools).
2. Files are uploaded directly by your team; nothing is fetched from the internet.
3. Example: a Maven hosted repo for internal libraries, with separate **release** and **snapshot** repositories.
4. Enforces that only approved, tested versions reach development and production.
5. Role-based access control restricts who can upload or download, and version history allows rollback.

### 4. Group Repositories

1. A group repository combines multiple repositories (proxy + hosted) behind **one URL**.
2. Developers configure only a single URL in `settings.xml` instead of many.
3. Nexus searches member repositories in a defined **order**, so priority can be controlled.
4. Adding or removing repositories later requires no client-side configuration changes.
5. Simplifies setup and reduces misconfiguration; commonly used as the main URL for builds.

### 5. Managing Docker Images

1. Nexus can act as a private Docker registry for storing and distributing images.
2. **Docker hosted repository:** stores internal, custom-built images securely.
3. **Docker proxy repository:** caches images from Docker Hub (e.g., `nginx`, `ubuntu`).
4. Caching speeds up pulls and helps avoid Docker Hub rate limits.
5. Developers and servers pull everything through Nexus, giving control, speed, and security.

### 6. Integration with CI/CD Pipelines

1. Nexus integrates with CI/CD tools like Jenkins, GitLab CI, GitHub Actions, and Azure DevOps to automate artifact handling.
2. Typical flow: code commit → Jenkins builds → artifact is pushed to Nexus → later stages pull it from Nexus.
3. In Jenkins, the **Nexus Artifact Uploader plugin** uploads build outputs (`.jar`, `.war`, etc.) to a hosted repository; you configure the Nexus URL, repository name, credentials, and artifact coordinates (`groupId`, `artifactId`, `version`).
4. Alternatively, Maven can deploy directly using `mvn deploy` with the `distributionManagement` section in `pom.xml`.
5. Credentials are stored securely in Jenkins (Credentials Manager), removing manual uploads and human error.

### 7. Snapshot and Release Management

1. **Snapshot** = work-in-progress build (version ends in `-SNAPSHOT`, e.g., `1.0-SNAPSHOT`), used during development.
2. **Release** = final, stable, tested version (e.g., `1.0`), intended for production.
3. Nexus keeps separate repositories for each, with different **version policies** (Snapshot vs Release).
4. Snapshots can be overwritten and updated repeatedly; releases are **immutable** (redeploy is disabled) to guarantee reproducible builds.
5. Production uses only release repositories, and cleanup policies can delete old snapshots to save storage.

### 8. Artifact Promotion

1. Promotion means moving an artifact through lifecycle stages: **Dev → Test/Staging → Production**.
2. The same, already-tested binary is promoted, not rebuilt, which guarantees what was tested is what gets released.
3. Example: after all tests pass, the artifact moves from a staging repository to a release repository.
4. Can be done manually, via REST API, or automated in the pipeline (Nexus Pro offers staging and promotion workflows; in OSS it is typically scripted).
5. Benefits: traceability, quality gate enforcement, and a clear audit trail of what reached production.

### 9. Security and Access Control

1. Nexus uses **RBAC (Role-Based Access Control)**: permissions are grouped into **privileges**, privileges into **roles**, and roles are assigned to users.
2. Example roles: **Developer** (read and deploy to snapshots), **Tester** (read-only on staging), **Admin** (full control).
3. Prevents unauthorized users from deploying to or modifying production release repositories.
4. Integrates with **LDAP/Active Directory** and SSO for centralized user management.
5. Additional security: HTTPS/SSL, API tokens, anonymous access control, and vulnerability scanning (Nexus IQ / Firewall).

### 10. Staging Repositories

1. A staging repository is a temporary area where artifacts are deployed **before** the official release.
2. Used for testing, QA, and approval without affecting production consumers.
3. Workflow: deploy to staging → run tests/approval → **promote** to release if passed, or **discard** if failed.
4. Protects the production repository from broken or unverified artifacts.
5. Provides a controlled, auditable release process; commonly seen when publishing to public repos like Maven Central.

### 11. Managing npm Packages

1. Nexus works as a private registry for npm (JavaScript/Node.js) packages, so teams don't depend only on npmjs.org.
2. **npm hosted repository:** stores internal packages your company publishes (e.g., `@company/ui-components`).
3. **npm proxy repository:** points to `https://registry.npmjs.org` and caches public packages after the first download.
4. **npm group repository:** combines hosted + proxy under one URL; developers set it once with `npm config set registry <nexus-group-url>`.
5. Publish with `npm publish --registry <hosted-url>`; authentication is handled using npm tokens or `.npmrc` credentials, and **scoped packages** (`@scope/name`) help avoid naming conflicts with public packages (dependency confusion).

### 12. Integration with Build Tools

1. Build tools (Maven, Gradle, npm, pip) are configured to **resolve dependencies from Nexus** and **deploy artifacts to Nexus**.
2. **Maven:** in `settings.xml`, set a `<mirror>` pointing to the Nexus **group** URL (with `mirrorOf` = `*`) and store credentials under `<servers>`.
3. **Maven deployment:** add `<distributionManagement>` in `pom.xml` with release and snapshot repository URLs, then run `mvn deploy`.
4. **Gradle:** declare `repositories { maven { url "<nexus-group-url>" } }` for resolving, and use the `maven-publish` plugin to publish.
5. Server IDs in `pom.xml` must match the `<server>` IDs in `settings.xml` for authentication to work; this is a common troubleshooting point (401/403 errors).

### 13. Artifact Cleanup Policies

1. Over time, repositories (especially snapshots and Docker images) consume large amounts of disk space.
2. **Cleanup policies** automatically remove old or unused artifacts based on criteria.
3. Common criteria: **last updated/published age** (e.g., older than 30 days), **last downloaded** (not used for X days), release type, and name/version patterns (regex).
4. Policies are created once, then **assigned to repositories**; they run as scheduled tasks.
5. Important: deleting components only marks them for removal; you must run the **"Compact blob store"** task to actually reclaim disk space.

### 14. Managing PyPI Packages

1. Nexus supports Python packages through PyPI repositories, used by `pip` and `twine`.
2. **PyPI hosted repository:** stores internal Python packages (wheels `.whl` and source distributions `.tar.gz`).
3. **PyPI proxy repository:** caches packages from `https://pypi.org`, speeding up installs and giving resilience if PyPI is down.
4. **PyPI group repository:** one URL for both; install with `pip install <package> --index-url <nexus-group-url>/simple`.
5. Upload internal packages using `twine upload --repository-url <hosted-url> dist/*`, with credentials configured in `~/.pypirc`.

### 15. Version Control for Artifacts

1. Nexus stores **multiple versions** of every artifact, so teams can reference, compare, and roll back to any version.
2. Uses **semantic versioning (SemVer):** `MAJOR.MINOR.PATCH` (e.g., `2.4.1`).
3. **MAJOR** = breaking changes, **MINOR** = new backward-compatible features, **PATCH** = backward-compatible bug fixes.
4. Version policy in Nexus (Release/Snapshot/Mixed) plus disabled redeploy on releases keeps versions **immutable** and reproducible.
5. Note: Nexus is not source control like Git; it version-stores **binaries (build outputs)**, while Git versions **source code**.

### 16. Backup and Restore

1. Nexus has two critical data parts: the **database** (metadata, config, users, repo settings) and the **blob stores** (the actual artifact files). Both must be backed up.
2. Schedule the **"Admin - Export databases for backup"** task (the database backup task), which writes a backup file of the DB to a folder you choose.
3. Back up the blob store directory separately (filesystem snapshot, rsync, or cloud storage snapshot), plus the `sonatype-work/nexus3` data directory and `etc/` config files.
4. **Restore:** stop Nexus, put the DB backup in the restore folder, restore the blob store files, and start Nexus. Run the **"Repair - Reconcile component database from blob store"** task if the DB and blob store are out of sync.
5. Best practices: take the DB backup **before** the blob store backup, store copies off-server, and **test restores regularly**. Exact steps vary by version and database type (embedded vs PostgreSQL).

### 17. Repository Health Check

1. Goal: continuously monitor the availability, performance, and quality of Nexus and its repositories.
2. **Repository Health Check (RHC)** analyzes components in a repository for known security vulnerabilities and license issues. Availability depends on the Nexus version and edition, so confirm it for the version you use.
3. Monitoring options include the **System Status** page, metrics, and the logs (`nexus.log`, `request.log`).
4. **Proxy repository status** shows if the remote is reachable; Nexus can automatically **block** an unreachable remote and retry later.
5. Also monitor disk space, blob store size, JVM memory, and thread usage, and expose metrics to tools like Prometheus/Grafana for alerting.

### 18. Integration with Security Scanners

1. Purpose: catch vulnerable or non-compliant components **before** they reach production.
2. The native Nexus solution is **Sonatype Nexus IQ Server (Lifecycle) and Nexus Firewall**: they scan components against vulnerability and license databases and can **quarantine or block** bad downloads at the proxy repository.
3. Note: **SonarQube** is a code-quality/SAST tool that analyzes **source code** in the CI stage; it does not scan artifacts stored in Nexus. Tools like **Trivy, Anchore, Snyk, or OWASP Dependency-Check** are commonly used for artifact and image scanning.
4. Typical pipeline: build → SonarQube (code analysis) → dependency/image scan → publish to Nexus → promote to release only if the policy passes.
5. Policies can fail the build on a threshold (for example, block on critical CVEs), which supports a **shift-left** security approach.

### 19. SSL/TLS Configuration

1. Purpose: encrypt traffic (HTTPS) between clients (Maven, npm, Docker, browsers) and Nexus, protecting credentials and artifacts.
2. **Option 1, direct:** configure Nexus's built-in Jetty server with a **Java keystore** (`keystore.jks`) containing the certificate and private key, enable the HTTPS port (commonly 8443) in `nexus.properties`, and enable the HTTPS connector in Jetty's config.
3. **Option 2, reverse proxy (common in production):** terminate SSL at **Nginx/Apache/load balancer** and forward requests to Nexus.
4. Docker registries **require HTTPS** (or must be explicitly configured as insecure), so TLS is essential for Docker repos.
5. If a **self-signed or internal CA** certificate is used, clients must trust it (import it into the Java truststore for Maven/Jenkins). Certificate expiry is a common outage cause, so monitor renewals.

### 20. Audit Logging

1. Audit logging records **who did what and when** in Nexus: logins, repository changes, user and role changes, uploads, deletions, and settings changes.
2. It is enabled via the **Audit capability** in the admin settings, and the log is viewable in the UI under the audit section.
3. Entries are also written to a log file (typically under `sonatype-work/nexus3/log/audit/audit.log`) in structured JSON format.
4. Supports **compliance and security** needs (SOX, ISO 27001, PCI-DSS) and helps with incident investigation and troubleshooting.
5. Best practice: forward logs to a central system (ELK/Splunk), protect them against tampering, and set retention and rotation.

### 21. Custom Repository Formats

1. Nexus 3 natively supports many formats (Maven, npm, Docker, PyPI, NuGet, RubyGems, Helm, Yum, APT, Go, Conan, and more), so a custom format is rarely needed.
2. For an organization-specific format, Nexus can be **extended with a plugin (bundle)** written in Java against the Nexus plugin API.
3. The plugin defines how the format stores, routes, and serves components (recipes, handlers, and metadata handling).
4. The built bundle is installed into the Nexus installation and appears as a new repository type in the UI.
5. Practical shortcut: use a **Raw (hosted) repository** to store arbitrary files (zips, binaries, configs) with no plugin. Custom plugins mean extra development and upgrade maintenance, so use them only when truly needed.

### 22. Dependency Management

1. Nexus does not decide *which* versions you use; it **stores and serves** the dependencies that your build tool resolves, making them consistent and available.
2. In **multi-module Maven projects**, define versions once in a **parent POM** or **BOM** using `<dependencyManagement>`, so every module uses the same versions.
3. All modules resolve through the Nexus **group repository** (via the `settings.xml` mirror), so everyone gets identical, cached artifacts.
4. Pin exact versions and avoid dynamic ones (`LATEST`, `1.+`, `RELEASE`) to keep builds **reproducible**; use lock files (`package-lock.json`, Gradle dependency locking).
5. Immutable releases in Nexus, plus proxy caching, mean a dependency can't change or vanish from upstream later; internal modules are published to the hosted repo for other modules to consume.

### 23. Cross-Region Artifact Distribution

1. Goal: give teams in different geographies **low-latency access** to artifacts and avoid slow cross-continent downloads.
2. Pattern: run a Nexus instance per region (for example, US, EU, APAC), with a primary instance where internal artifacts are published.
3. **Option A:** regional instances use the primary as a **proxy repository**, so artifacts are fetched on first request and cached locally.
4. **Option B:** replicate content proactively using Nexus's replication features (these are in the commercial/Pro edition, so check your version), or scripts via the **REST API**; for cloud blob stores, cloud-native replication such as **S3 cross-region replication** can help.
5. Trade-offs: replication lag (eventual consistency), extra storage cost, and the need to keep security, cleanup, and versions consistent across regions.

### 24. LDAP Integration

1. LDAP lets Nexus **authenticate users against a central directory** (OpenLDAP, Active Directory), so users sign in with corporate credentials instead of separate Nexus accounts.
2. Configure under **Administration → Security → LDAP**: server host, port, protocol (**use LDAPS**), search base DN, and a **system (bind) account** for lookups.
3. Set **user mapping** (user base DN, object class, user ID attribute, email attribute) and **group mapping** (static groups or dynamic via `memberOf`).
4. Under **Roles**, map **LDAP groups to Nexus roles** (external role mapping), so permissions follow group membership; add the **LDAP Realm** to the active realms in Security → Realms.
5. Benefits: centralized onboarding and offboarding, one password policy, and easier audits. Keep a local **admin** account as a fallback if LDAP is down, and verify with the "Verify login" test.

### 25. High Availability Configuration

1. HA means no single point of failure: if one Nexus node fails, others keep serving artifacts.
2. HA is a **Nexus Repository Pro** capability; the OSS edition runs as a single node, so teams use fast backup/restore or failover plans instead.
3. Typical architecture: **multiple Nexus nodes** behind a **load balancer**, an **external PostgreSQL database** (itself highly available), and a **shared blob store** (such as S3 or Azure Blob).
4. Commonly deployed on **Kubernetes** or cloud platforms (for example, AWS with auto-scaling and multiple availability zones); exact requirements differ by version, and older HA setups used different clustering.
5. Also plan for: health-check endpoints for the load balancer, shared storage performance, license coverage, and rolling upgrades. HA is **not** a replacement for backups.
