### 26. Custom Metadata Management

1. Goal: attach useful tracking information (build number, changelog, release notes, commit ID) to artifacts so they can be identified and audited later.
2. Nexus automatically records **asset attributes**, such as checksums (SHA-1, SHA-256, MD5), upload time, and the uploading user, and it reads format metadata (`pom.xml` for Maven, `package.json` for npm, labels for Docker).
3. **Tags** (available via the UI/REST API, with some features edition-dependent) can be applied to components, for example a build number or `release-candidate`, and used later to search, filter, or promote.
4. For notes, changelogs, or SBOMs, upload them as **additional assets** next to the main artifact (for example with a Maven classifier like `1.0.0-changelog.txt`) or in a Raw repository.
5. Practical conventions that work everywhere: put build info in **Docker labels**, in the version string, or in a metadata file published alongside the artifact. Nexus is not a full metadata database, so keep detailed records in your CI system too.

### 27. Artifact Provenance

1. Provenance means being able to prove **where an artifact came from**: which source commit, which build, which pipeline, and who published it.
2. Nexus helps by recording who uploaded each asset and when, the **checksums** for integrity, and, for proxy repositories, which remote source the component came from.
3. Strengthen it by **signing artifacts** (GPG `.asc` for Maven, `cosign` for container images) so consumers can verify authenticity.
4. Publish the **build information** (Jenkins build number, Git commit SHA, build URL) with the artifact, and generate an **SBOM** (Software Bill of Materials, e.g. CycloneDX) listing all the components inside it.
5. This supports **compliance and supply-chain security** (frameworks such as SLSA), plus audit logs for investigating incidents. Together these allow you to trace any production artifact back to its source.

### 28. REST API for Automation

1. Nexus 3 provides a **REST API** under `/service/rest/v1/`, with built-in **Swagger UI** documentation in the admin interface (API reference page).
2. It can manage **repositories** (create, update, delete), **components and assets** (upload, search, delete), **users, roles, privileges**, blob stores, and scheduled **tasks**.
3. Example upload:

       curl -u user:pass -X POST "<nexus>/service/rest/v1/components?repository=maven-releases" -F maven2.groupId=com.acme -F maven2.artifactId=app -F maven2.version=1.0 -F maven2.asset1=@app.jar -F maven2.asset1.extension=jar

4. Example create repository: `POST /service/rest/v1/repositories/maven/hosted` with a JSON body; this is ideal for **infrastructure-as-code** setups (scripts, Ansible, Terraform).
5. Use a **dedicated service account** with minimum privileges (or API tokens) instead of admin credentials, and use HTTPS. The older Groovy scripting API is disabled by default in recent versions for security reasons.

### 29. Docker Image Scanning

1. Purpose: find **known vulnerabilities (CVEs)** in OS packages and libraries inside Docker images before they run in production.
2. Nexus stores the images; the scanning is done by **integrated tools** such as Sonatype Nexus IQ/Lifecycle, Trivy, Grype, Anchore, Clair, or Snyk.
3. Typical pipeline: build image → **scan** → fail the pipeline on critical findings → push to the Nexus Docker hosted repo → promote after it passes.
4. For public images cached through a **Docker proxy** repository, Nexus Firewall can quarantine risky images, or you scan on pull from Nexus.
5. Best practices: use minimal base images, **rescan regularly** (new CVEs appear for old images), and sign images. Note that SonarQube analyzes source code, not Docker images.

### 30. Artifact Lifecycle Policies

1. Goal: control an artifact's journey through **development → testing/staging → production**, so only approved artifacts reach production.
2. Nexus has no single "lifecycle policy" switch; it is achieved by **combining features**, which is a good point to state in an interview.
3. **Separate repositories per stage** (snapshot, staging, release) plus **promotion** of the same tested binary between them.
4. **RBAC** controls who can deploy or promote, **cleanup policies** remove old snapshots and unused artifacts, and **security/policy scanning** (Nexus IQ) acts as a gate before promotion.
5. Outcome: a controlled, auditable path with **immutable releases**, less storage waste, and a clear record of what was approved and when.

### 31. Repository Replication

1. Replication copies artifacts from one Nexus instance to another, so teams in different locations (or a disaster recovery site) have the same content available locally.
2. Typical setup: a **primary** Nexus where artifacts are published, and one or more **secondary** instances that receive the content.
3. Ways to do it: the **built-in replication or Smart Proxy features** (commercial/Pro edition, so confirm for your version), a secondary Nexus configured as a **proxy repository** of the primary, or custom scripts using the **REST API**.
4. Cloud-level options also exist, such as **S3 cross-region replication** when blob stores are on S3.
5. Watch for: **replication lag** (eventual consistency), network bandwidth, storage cost, one-way vs two-way sync (conflicts), and keeping security and cleanup rules consistent across instances.

### 32. Environment-Specific Repositories

1. Idea: use separate repositories for each stage, such as **dev, test/staging, and prod**, to isolate artifacts and avoid mixing unverified code with approved code.
2. Dev usually uses **snapshot** repositories (mutable); test and prod use **release** repositories (immutable).
3. Create a **group repository per environment**; for example, the dev group contains snapshots + releases + proxy, while the prod group contains **only approved releases** + proxy.
4. Use **RBAC** so developers can deploy to dev, CI can deploy to test, and only a controlled pipeline or admin can write to prod.
5. Use clear naming (for example `maven-dev`, `maven-test`, `maven-prod`), and **promote the same artifact** between environments instead of rebuilding; keep environment-specific configuration outside the artifact (config files, environment variables).

### 33. Continuous Integration (CI)

1. CI means every code commit triggers an **automated build and test**; Nexus is where the resulting artifacts are stored and fetched from.
2. Typical Jenkins pipeline: **checkout → build (`mvn clean package`) → unit tests → code analysis (SonarQube) → publish to Nexus (`mvn deploy` or the Nexus plugin) → deploy to an environment**.
3. Builds **resolve dependencies from the Nexus group repository**, which makes them faster and consistent.
4. Jenkins plugins: **Nexus Artifact Uploader** (simple uploads) and the **Sonatype Nexus Platform plugin** (more features, including policy evaluation); credentials are kept in Jenkins' credential store.
5. The same concept works with GitLab CI, GitHub Actions, and Azure DevOps. Nexus **webhooks** can notify other systems (for example, trigger a deploy job) when a new component is published.

### 34. Artifact Promotion Automation

1. Goal: move artifacts automatically from **dev → test → prod** when predefined criteria are met, without manual copying.
2. Typical criteria: **tests passed**, **security/vulnerability scan passed**, code quality gate passed, and sometimes a **manual approval** step.
3. Implementation options: a Jenkins/GitLab stage that calls the Nexus **REST API** to copy or move the component; with Nexus Pro, use **tags and staging move** features; or a script that downloads from one repository and uploads to the next.
4. Always promote the **exact same binary** and verify its **checksum** after the copy, then log who or what triggered the promotion.
5. Keep an **approval gate** (for example Jenkins `input` step) before production, and use a service account with limited permissions. Automation gives speed, consistency, and an audit trail.

### 35. Blob Store Management

1. A **blob store** is the actual storage location for artifact files on disk or in the cloud (the database only holds metadata). Each repository is assigned to one blob store.
2. Types include **File** (local or network disk) and cloud options such as **S3**, and (depending on edition/version) Azure or Google Cloud storage.
3. Use **multiple blob stores** to separate workloads, for example one for large Docker images, one for Maven, one for proxy caches, and a fast SSD for heavily used repositories.
4. Set **soft quotas** (space used or file count) and alerts, and monitor disk usage so it doesn't fill up unexpectedly.
5. Run **cleanup policies** plus the **"Compact blob store"** task to reclaim space, and include each blob store in your **backup plan**. Choose the blob store at repository creation, because changing it later is limited and depends on the edition.

### 36. Data Retention Policies

1. Retention policies decide **how long artifacts are kept** before automatic deletion, to save storage and meet data-management or compliance rules.
2. Implemented with **cleanup policies** (by age, last downloaded, release type, name/version pattern) assigned to repositories and run as scheduled tasks.
3. Related tasks: **Remove snapshots** (keep the latest N or delete by age), **delete unused Docker manifests/images**, and **delete incomplete uploads**.
4. Releases are normally kept longer (or forever), while snapshots and cached proxy content are cleaned aggressively; some legal or audit rules require fixed retention periods.
5. Deleted content only frees space after the **"Compact blob store"** task runs; where available, preview what a policy will delete before enabling it.

### 37. Artifact Verification

1. Verification proves a file has **not been corrupted or altered**, using checksums (MD5, SHA-1, SHA-256, SHA-512).
2. Nexus calculates checksums when an artifact is stored; Maven repositories also hold `.sha1`/`.md5` files, and clients compare them on download.
3. Manual check: compute `sha256sum app.jar` and compare it with the value in Nexus; Nexus search can also find a component **by checksum**, which helps identify an unknown JAR.
4. Maven can be set to fail on bad checksums (strict mode), which prevents corrupted downloads from entering a build.
5. Checksums prove **integrity** only; to prove **authenticity** (who produced it), you need signatures (topic 42).

### 38. Repository Browsing

1. The Nexus web UI has a **Browse** section to explore repositories, folders, components, and assets in a tree view.
2. **Search** supports keywords, group/artifact/version, format, and checksums, and it can be filtered by repository.
3. Component details show versions, file paths, checksums, upload time, and **ready-to-copy dependency snippets** (for example, a Maven `<dependency>` block).
4. Users can download files directly from the UI; what each user can see depends on their **RBAC privileges** (browse/read), and anonymous access can be enabled or disabled.
5. The same search and listing can be done through the **REST API** for scripts and automation.

### 39. Storage Quotas

1. Quotas prevent a repository's storage from growing uncontrolled and filling the disk.
2. In Nexus, quotas are set at the **blob store** level as a **soft quota**: a limit on space used or minimum space remaining.
3. **Important:** a soft quota **alerts** (the blob store shows a failing health state and can be monitored) but does **not block uploads**.
4. To limit a specific repository, give it its **own blob store** and set the quota on it.
5. For hard limits, rely on **filesystem or volume size limits**, plus cleanup policies, compaction, and monitoring.

### 40. Build Promotion

1. Build promotion moves the **entire output of a successful build** (all artifacts plus metadata) to the next stage, typically a release repository.
2. Flow: CI builds → publishes to a snapshot or staging repository → automated tests and scans run → on success, the build is promoted to the release repository.
3. Promotion criteria: all tests pass, quality gate passes, security scan is clean, and optionally a manual approval.
4. It uses the same mechanisms as artifact promotion (REST API, Pro staging/tags, or pipeline scripts, topics 8 and 34).
5. Key principle: **promote, don't rebuild**, so the tested binary is exactly what reaches production; keep the build number for traceability.

### 41. Repository Mirroring

1. Goal: keep fast, reliable access to critical external dependencies even when the remote site is slow or down.
2. In Nexus, mirroring is done by a **proxy repository** that caches content on demand (not a full copy of the remote site).
3. On the client side, Maven's `<mirror>` setting in `settings.xml` redirects all requests to the Nexus group URL.
4. Tuning options: **content/metadata max age**, **negative cache** (remember "not found"), and **auto-block** when the remote is unreachable.
5. A full mirror (copying everything) is rarely done because of size; instead, cached content plus backups is the usual approach.

### 42. Artifact Signing

1. Signing proves **authenticity and integrity**: the artifact really comes from you and hasn't been changed.
2. For Maven: generate a **GPG key pair**, use the `maven-gpg-plugin` to create `.asc` signature files, and upload them along with the artifact (required for Maven Central).
3. Consumers verify with the **public key** (for example `gpg --verify app.jar.asc app.jar`).
4. For container images, tools such as **cosign** or Docker Content Trust are used; Nexus stores the signature artifacts but does not sign for you.
5. Protect the private key (use Jenkins credentials or a secrets manager, never source control), rotate keys, and don't confuse signatures with checksums.

### 43. Repository Backup Automation

1. Automation means backups run **on a schedule** without manual action, supporting disaster recovery.
2. Use the Nexus scheduled task **"Admin - Export databases for backup"** with a cron schedule (or database-native backup tools such as `pg_dump` when using PostgreSQL).
3. Blob stores are backed up with external jobs: cron + `rsync`, storage snapshots, or cloud versioning/replication.
4. Define **retention** for the backups, copy them **off-site**, and set alerts if a backup job fails.
5. **Test restores regularly** and track your RPO (how much data you can lose) and RTO (how fast you must recover).

### 44. Continuous Delivery (CD)

1. In CD, Nexus is the **trusted source** from which deployment tools fetch the exact artifact version to release.
2. Flow: tested artifact in the Nexus release repo → deployment tool (Jenkins, Ansible, Helm, Argo CD, Kubernetes) pulls it → deploys to the environment.
3. Always deploy a **specific immutable version**, never "latest", so deployments are repeatable and **rollback** is simply redeploying the previous version.
4. Docker images and Helm charts can also be pulled from Nexus.
5. Terminology: **Continuous Delivery** keeps the release ready, usually with a manual approval; **Continuous Deployment** pushes to production automatically.

### 45. Version Conflict Management

1. A conflict occurs when different dependencies require **different versions of the same library**.
2. Important: **Nexus doesn't resolve conflicts; the build tool does** (Maven uses "nearest definition wins"; Gradle picks the highest version by default).
3. Nexus helps by serving consistent, immutable versions and allowing **search** to see which versions exist (Nexus IQ can also show usage and risk).
4. Tools to find and fix conflicts: `mvn dependency:tree`, `mvn dependency:analyze`, Gradle `dependencyInsight`, the Maven **Enforcer** plugin (dependency convergence), exclusions, and **BOMs**.
5. Prevent them with a parent POM/BOM, pinned versions, and lock files.

### 46. Private Docker Registry

1. Nexus can be your company's **private Docker registry** for internal images, with access control and security.
2. Create a **Docker hosted repository** with its own HTTP/HTTPS connector port (or use subdomain/path routing behind a reverse proxy).
3. Workflow: `docker login <nexus-host:port>` → `docker tag image <nexus-host:port>/app:1.0` → `docker push` → `docker pull`.
4. Enable the **Docker Bearer Token realm**, use **HTTPS** (Docker expects it), and combine hosted + proxy into a **group** for a single pull URL.
5. Control growth with cleanup policies for unused manifests and a dedicated blob store, as Docker layers are large.

### 47. Custom Artifact Repositories

1. For proprietary or niche file types, first use a **Raw repository** (hosted, proxy, or group), which stores any file by path.
2. Typical Raw uses: installers, ZIP bundles, firmware, configuration packages, and scripts.
3. Use a clear **path convention** (for example `/product/version/file`) since Raw has no built-in version metadata.
4. If format-specific behavior is truly needed, develop a **Nexus plugin** (topic 21), which takes more effort and upgrade maintenance.
5. Apply the usual controls: RBAC, cleanup policies, and backups.

### 48. Resource Optimization

1. Monitor **CPU, memory (JVM heap), disk space, disk I/O, and file descriptors**, the main drivers of Nexus stability.
2. Tune the JVM in `nexus.vmoptions`: set `-Xms` equal to `-Xmx` and size `MaxDirectMemorySize` properly, following Sonatype's sizing guidance for your version.
3. Use fast disks (SSD) for the database and raise the OS file descriptor limit (`ulimit -n`).
4. Run cleanup and compact tasks **off-peak**, and keep repositories lean to reduce load.
5. Monitor with the status page, logs, and a **Prometheus/Grafana** metrics endpoint; add alerts for disk, heap, and response time, and scale up or move to HA when needed.

### 49. Artifact Archiving

1. Archiving moves rarely used old artifacts to **cheaper long-term storage**, reducing load on primary storage.
2. Nexus has **no built-in archive tier**, so it is done with processes around it.
3. Common approach: export old artifacts via the **REST API** to cheap storage (object storage or NAS), record what was archived, then remove them with cleanup policies.
4. Alternative: keep an **"archive" hosted repository** on a cheaper blob store for old releases.
5. Caution: don't move blob files to cold storage underneath Nexus (they become unreadable), keep a documented **restore process**, and check compliance retention rules first.

### 50. Cross-Team Collaboration

1. Nexus gives all teams a **shared, trusted place** for libraries and dependencies, which improves reuse and consistency.
2. Shared internal libraries are published once to a hosted repository, and every team consumes them through the **same group URL**.
3. Use **RBAC and content selectors** (path/coordinate-based rules) so each team can manage its own area without touching others'.
4. Agree on **naming conventions and SemVer** so versions are predictable, and document each library's ownership.
5. A central **platform/DevOps team** usually owns Nexus standards (cleanup, security scanning, backups), and this approach also supports inner-source practices.
