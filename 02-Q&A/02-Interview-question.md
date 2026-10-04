# NEXUS REPOSITORY 3 — TOP 10 SCENARIO-BASED INTERVIEW QUESTIONS & ANSWERS

## 1. DEVELOPERS CANNOT DOWNLOAD A DEPENDENCY FROM NEXUS

**Q: A developer runs "mvn clean install" but gets:**

    "Could not find artifact". How would you troubleshoot?

### Answer:

I would troubleshoot it step by step instead of assuming Nexus is broken.

1. Check the dependency coordinates:
   - Group ID
   - Artifact ID
   - Version

2. Check Maven settings.xml:
   - Is Maven pointing to the correct Nexus URL?
   - Are the credentials correct?
   - Is the correct repository configured?

3. Check Nexus:
   - Does the artifact actually exist?
   - Is it in a hosted, proxy, or group repository?

4. Check permissions:
   - Does the user have Browse/Read access?
   - Check User -> Role -> Privileges -> Repository.

5. If it is a proxy repository:
   - Check the remote/upstream URL.
   - Check whether Nexus can reach the upstream repository.
   - Check proxy/cache status.

6. Check Nexus logs and Maven logs.

### Common HTTP errors:

    401 = Authentication problem
    403 = Permission problem
    404 = Artifact not found
    5xx = Server/upstream problem

### Interview answer:

> "I would first verify the dependency coordinates and Maven settings.xml. Then I would check whether the artifact exists in Nexus and whether the user has Read permission. If it is a proxy repository, I would verify the upstream URL, network connectivity, cache, and SSL configuration. Finally, I would check Nexus and Maven logs to identify the exact root cause."

---

## 2. NEXUS IS RUNNING OUT OF DISK SPACE

**Q: Nexus disk usage has reached 95% and builds are failing. What would you do?**

### Answer:

I would not manually delete files from the Nexus filesystem.

1. Identify which Blob Store is consuming the most space.

### Example:

    maven-releases  -> 20 GB
    maven-snapshots -> 500 GB
    docker          -> 200 GB

2. Check which repositories contain old or unnecessary artifacts.

3. Check cleanup policies.

For example:

    - Delete old SNAPSHOTs.
    - Delete unused/old components based on retention requirements.

4. Be careful with release artifacts because production applications may depend on them.

5. Run the appropriate cleanup process.

6. Monitor storage after cleanup.

7. If storage is still insufficient, plan additional storage capacity.

### Important:

Do not simply delete files directly from the Blob Store because Nexus maintains metadata and storage relationships.

### Interview answer:

> "I would first identify the Blob Store consuming the most space. Then I would review snapshots and old artifacts and use cleanup policies rather than manually deleting files. I would protect production release artifacts, run cleanup, monitor the storage, and increase capacity if required."

---

## 3. NEXUS PROXY CANNOT DOWNLOAD FROM MAVEN CENTRAL

**Q: A Nexus proxy repository cannot download packages from Maven Central. How would you troubleshoot it?**

### Answer:

I would follow the complete communication path:

    Nexus
      |
      v
    Proxy Repository
      |
      v
    Remote URL
      |
      v
    Network / Corporate Proxy
      |
      v
    Maven Central

I would check:

1. Remote URL:
   - Is the upstream URL correct?

2. Network connectivity:
   - Can the Nexus server reach the internet?
   - Check firewall rules.
   - Check DNS.
   - Check routing.

3. Corporate proxy:
   - Does Nexus need to use an enterprise HTTP/HTTPS proxy?
   - Are proxy settings correct?

4. Authentication:
   - Does the upstream repository require credentials?

5. SSL/TLS:
   - Check certificates.
   - Check trust configuration.
   - Look for errors such as:
     - "PKIX path building failed"
     - "SSL handshake failed"

6. Repository health/status.

7. Nexus logs.

### Interview answer:

> "For a proxy failure, I would first verify the remote URL and then check network connectivity, DNS, firewall, and corporate proxy configuration. I would also verify upstream authentication and SSL certificates. Finally, I would inspect Nexus logs to determine whether the issue is network, authentication, SSL, or upstream related."

---

## 4. USER CAN DOWNLOAD BUT CANNOT UPLOAD

**Q: A developer can download artifacts from Nexus but cannot upload them. What could be wrong?**

### Answer:

This is usually a permissions issue.

Downloading and uploading require different privileges.

### Typical privileges:

    Browse -> See/search artifacts
    Read   -> Download artifacts
    Add    -> Upload artifacts
    Edit   -> Modify artifacts
    Delete -> Remove artifacts

The user may have:

    Browse + Read

but not:

    Add

I would check:

    User
      |
      v
    Role
      |
      v
    Privileges
      |
      v
    Repository

I would also check whether a Content Selector is restricting the user.

### For example:

    Developer
       |
       +-- Read
       +-- Browse
       +-- No Add

Therefore:

    Download -> SUCCESS
    Upload   -> FAILURE

### Interview answer:

> "I would check the user's roles and privileges. Read permission allows downloading, while Add permission is required for uploading. I would verify that the user has Add access to the target hosted repository and that no content selector is blocking the upload."

---

## 5. TWO TEAMS NEED DIFFERENT ARTIFACT ACCESS

**Q: Team A and Team B use the same repository, but Team A should only access project-a and Team B should only access project-b. How would you implement this?**

### Answer:

I would use:

    Roles + Privileges + Content Selectors

### Example:

Repository:

    company-repository

Artifacts:

    project-a/*
    project-b/*
    project-c/*

Create a content selector for Team A:

    project-a/*

Create another content selector for Team B:

    project-b/*

Then assign privileges through roles:

    Team A Role
       |
       +-- Read project-a/*

    Team B Role
       |
       +-- Read project-b/*

This provides fine-grained access control.

I would follow the principle of least privilege.

That means:

    Team A should NOT get access to everything.
    Team B should NOT get access to everything.

### Interview answer:

> "I would use content-selector-based privileges with roles. Team A would receive access only to project-a paths, while Team B would receive access only to project-b paths. This gives fine-grained access and follows the principle of least privilege."

---

## 6. CI/CD PIPELINE BUILDS SUCCESSFULLY BUT CANNOT UPLOAD TO NEXUS

**Q: Jenkins/GitLab CI successfully builds the application but fails while uploading the artifact to Nexus. How would you troubleshoot it?**

### Answer:

The pipeline is:

    Git
      |
      v
    Build
      |
      v
    Test
      |
      v
    Upload to Nexus  <-- FAILURE

I would check the following:

1. Nexus URL:
   - Is the CI/CD pipeline using the correct repository URL?

2. Credentials:
   - Are the Nexus username/token credentials correct?
   - Have they expired?

3. Permissions:
   - Does the CI service account have Add permission?

4. Repository type:
   - Is the target repository a hosted repository?
   - Is the pipeline accidentally trying to upload to a proxy/group repository?

5. Version:
   - Is the artifact a RELEASE or SNAPSHOT?

### Example:

    1.2.0-SNAPSHOT
        |
        v
    Snapshot Repository

A release artifact should normally go to the release repository.

6. Deployment policy:
   - Is redeployment allowed?
   - Does the artifact already exist?

### Example:

    myapp-1.0.0.jar already exists
                |
                v
           Redeploy blocked

7. Check Nexus and CI/CD logs.

### Interview answer:

> "I would first check the Nexus URL and CI credentials. Then I would verify that the service account has Add permission on the target hosted repository. I would check whether the artifact is a release or snapshot and whether the repository's deployment policy allows the upload or redeployment. Finally, I would check the CI and Nexus logs."

---

## 7. DOCKER PUSH WORKS BUT DOCKER PULL FAILS

**Q: A Docker image can be pushed to Nexus but cannot be pulled. What would you check?**

### Answer:

Since push works, authentication and connectivity are probably partially working.

I would check:

1. Read permission:
   - Push requires write/add access.
   - Pull requires read access.

2. Docker login:
   - Is the user authenticated to the correct Nexus Docker registry?

3. Image name:

### Example:

    nexus.company.com/myapp:1.0

Make sure the repository path and image name are correct.

4. Does the image actually exist in Nexus?

5. Docker repository configuration:
   - Check Docker connector/port.
   - Check HTTPS configuration.

6. SSL/TLS certificate:
   - Docker may reject an invalid or untrusted certificate.

7. Check Nexus logs.

### Important idea:

    Push -> Write/Add
    Pull -> Read

### Interview answer:

> "Since the push is successful, I would first verify Read permission for the user or service account. Then I would check the image name, tag, repository path, Docker login, connector configuration, and SSL/TLS setup. I would also confirm that the image exists in the expected Docker repository."

---

## 8. BUILD WORKS ON ONE MACHINE BUT FAILS ON ANOTHER

**Q: Maven build works on Developer A's machine but fails on Developer B's machine. How would you troubleshoot it?**

### Answer:

I would compare both environments.

Check:

1. Java version:

### Example:

    Developer A -> Java 17
    Developer B -> Java 11

2. Maven version.

3. Maven settings.xml:
   - Same Nexus URL?
   - Same mirror?
   - Same repository?
   - Correct credentials?

4. User permissions:
   - Does Developer B have access to Nexus?

5. Local Maven cache:

    Developer A:
       ~/.m2/repository
           |
           +-- Dependency already cached

    Developer B:
       Dependency not cached
           |
           v
       Needs Nexus access

6. Network:
   - VPN?
   - Firewall?
   - Corporate proxy?
   - DNS?

7. Compare the exact error messages.

### Important point:

Sometimes Developer A appears to work because the dependency is already available in the local Maven cache, while Developer B actually needs to download it from Nexus.

### Interview answer:

> "I would compare Java and Maven versions, settings.xml, Nexus credentials, permissions, local Maven cache, and network configuration between the two machines. A common reason is that the working machine already has the dependency cached locally, while the failing machine needs to retrieve it from Nexus."

---

## 9. A RELEASE ARTIFACT WAS ACCIDENTALLY OVERWRITTEN

**Q: Someone accidentally overwrote a released artifact. How would you prevent this?**

### Answer:

Released artifacts should normally be immutable.

### Example:

    myapp-2.0.0.jar
          |
          v
        RELEASE
          |
          v
    Should NOT be overwritten

Configure the release repository so that redeployment of an existing version is not allowed.

Instead of replacing:

    myapp-2.0.0

publish a new version:

    myapp-2.0.1

This gives:

    Reproducibility
    Traceability
    Reliable deployments
    Safer rollback

### Best practice:

    Build
      |
      v
    Test
      |
      v
    Release
      |
      v
    Immutable Artifact
      |
      v
    Deploy

### Important interview phrase:

"Build once and promote the same artifact."

Do not rebuild a different binary for every environment.

### Interview answer:

> "I would configure the release repository to prevent redeployment of an existing version and enforce immutable release artifacts. If a change is required, we should publish a new version instead of overwriting the existing artifact."

---

## 10. CI/CD BUILDS HAVE SUDDENLY BECOME VERY SLOW

**Q: Yesterday a build took 5 minutes, but today it takes 30 minutes. Nexus is involved. How would you troubleshoot it?**

### Answer:

I would first identify WHERE the time is being spent.

The flow is:

    CI/CD
       |
       v
    Nexus
       |
       v
    Proxy
       |
       v
    Upstream Repository

### Possible causes:

1. Nexus cache miss:

    CI
     |
     v
    Nexus
     |
     v
    Cache miss
     |
     v
    Internet
     |
     v
    Upstream Repository

The package has to be downloaded again.

2. Upstream repository is slow.

    Nexus ---> Maven Central
                   |
                   v
                 Slow

3. Network problem:
   - High latency
   - DNS problem
   - Firewall
   - Corporate proxy
   - Packet loss

4. Nexus performance:
   - High CPU
   - High memory usage
   - High disk I/O
   - Too many concurrent requests

5. Blob Store performance:
   - Slow storage
   - Storage capacity problems
   - I/O bottleneck

6. CI runner problem:
   - Jenkins/GitLab runner overloaded
   - Network issue
   - Container/resource problem

7. Check Nexus logs and CI logs.

I would compare:

    Previous build:
       5 minutes

    Current build:
       30 minutes

Then identify which stage became slower.

### Interview answer:

> "I would first identify whether the slowdown is between the CI runner and Nexus, between Nexus and the upstream repository, or inside the build itself. I would check cache hits, upstream response time, network latency, Nexus CPU and memory, disk I/O, Blob Store performance, and Nexus logs. I would also check whether the CI runner itself is overloaded."

---

## BONUS: GENERAL NEXUS TROUBLESHOOTING FORMULA

For almost any Nexus problem, follow this order:

    1. CLIENT
       |
       | Is Maven/npm/Docker configured correctly?
       v
    2. URL
       |
       | Is the correct Nexus repository being used?
       v
    3. AUTHENTICATION
       |
       | Is the user/token valid?
       v
    4. AUTHORIZATION
       |
       | Does the user have the required privileges?
       v
    5. REPOSITORY
       |
       | Hosted / Proxy / Group?
       | Correct policy?
       v
    6. NETWORK
       |
       | DNS / Firewall / Proxy / Connectivity?
       v
    7. UPSTREAM
       |
       | Is the external repository available?
       v
    8. STORAGE
       |
       | Blob Store / Disk / I/O?
       v
    9. LOGS
       |
       | What exact error is Nexus reporting?
       v
    10. ROOT CAUSE
        |
        +--> Fix
        +--> Test
        +--> Monitor

---

## IMPORTANT HTTP STATUS CODES TO REMEMBER

    401 -> Authentication failed
    403 -> User authenticated but does not have permission
    404 -> Artifact/resource not found
    408 -> Request timeout
    429 -> Too many requests/rate limiting
    500 -> Internal server error
    502 -> Bad gateway/upstream problem
    503 -> Service unavailable
    504 -> Gateway timeout

---

## IMPORTANT NEXUS CONCEPTS TO MENTION IN INTERVIEWS

### Hosted Repository

    -> Stores your organization's artifacts.

### Proxy Repository

    -> Downloads and caches artifacts from an external repository.

### Group Repository

    -> Combines multiple repositories behind one URL.

### Blob Store

    -> Stores the actual artifact files.

### Cleanup Policy

    -> Removes old/unwanted artifacts according to defined rules.

### Roles

    -> Group permissions for users.

### Privileges

    -> Define what actions users can perform.

### Content Selectors

    -> Provide fine-grained access to specific artifact paths.

### Repository Policy

    -> Controls release/snapshot behavior.

### Deployment Policy

    -> Controls whether artifacts can be redeployed/overwritten.

### Proxy Cache

    -> Stores downloaded external artifacts locally.

### Immutable Artifact

    -> Released artifact should not be changed after publication.

### CI/CD Integration

    -> Build -> Nexus -> Deploy.

### Artifact Promotion

    -> Move the same tested artifact through environments.

### Logs

    -> Help identify the root cause of Nexus problems.

## 11. NEXUS SERVER IS COMPLETELY DOWN IN PRODUCTION

**Q: Developers and CI/CD pipelines cannot access Nexus at all. The Nexus UI is also not opening. What would you do?**

### Answer:

I would treat this as a production incident and first determine whether the problem is with Nexus itself, the server, the network, or the load balancer.

### Step 1: Check availability

    Client
      |
      v
    Load Balancer
      |
      v
    Nexus Server

Check:

- Can the Nexus URL be resolved?
- Is DNS working?
- Can the server be reached?
- Is the Nexus service running?

### Step 2: Check the Nexus process/service.

If the service is stopped, check why it stopped.

### Step 3: Check server resources:

    CPU
    Memory
    Disk
    Disk I/O

A full disk or exhausted memory can cause Nexus problems.

### Step 4: Check Nexus logs.

Look for:

- OutOfMemoryError
- Disk full
- Database/configuration errors
- Blob Store errors
- Startup failures

### Step 5: Check reverse proxy/load balancer.

Sometimes Nexus is healthy but the load balancer or reverse proxy is failing.

### Step 6: If there was a recent change, investigate it.

For example:

- Certificate change
- Network change
- Firewall change
- Nexus configuration change
- OS patching

### Step 7: If recovery is not possible, follow the disaster recovery procedure.

### Interview answer:

> "I would first determine whether the failure is at the DNS, load balancer, server, or Nexus application layer. I would check the Nexus service, CPU, memory, disk, disk I/O, and logs. I would also verify the load balancer and network path. If the issue cannot be recovered quickly, I would follow the documented backup and disaster recovery procedure."

---

## 12. NEXUS IS RUNNING BUT USERS ARE GETTING 503 ERRORS

**Q: Nexus is running, but users are receiving HTTP 503 Service Unavailable. What would you check?**

### Answer:

503 generally means the service is temporarily unavailable.

I would check the complete request path:

    User
      |
      v
    Load Balancer
      |
      v
    Reverse Proxy
      |
      v
    Nexus
      |
      v
    Storage

First check whether Nexus itself is healthy.

Then check:

1. Load balancer health checks.
2. Reverse proxy configuration.
3. Nexus application health.
4. CPU and memory.
5. Disk availability.
6. Blob Store availability.
7. Nexus logs.
8. Recent infrastructure changes.

If Nexus is healthy but the load balancer reports it as unhealthy, the problem may be outside Nexus.

### Interview answer:

> "I would not assume that 503 means Nexus itself is down. I would check the load balancer and reverse proxy first, then verify the Nexus application health, system resources, storage, and logs. This helps identify which layer is actually returning the 503."

---

## 13. NEXUS DISK IS 100% FULL

**Q: Production Nexus has reached 100% disk usage. What would you do immediately?**

### Answer:

This is a critical production issue.

First:

    STOP making unnecessary changes.

Then identify:

    Which filesystem is full?
    Which Blob Store is consuming space?
    Is the Nexus process still healthy?

Check:

    df -h
    df -i

Also check inode usage because a filesystem can run out of inodes even when there appears to be free disk space.

Then identify what is consuming space.

### Possible causes:

- Old snapshots
- Docker images
- Large artifacts
- Logs
- Temporary files
- Backup files

Do NOT manually delete Nexus Blob Store files.

Use the appropriate Nexus cleanup process.

If production is severely impacted, increase storage capacity according to the organization's emergency procedure.

After recovering space:

1. Verify Nexus health.
2. Test artifact download.
3. Test artifact upload.
4. Review cleanup policies.
5. Set storage alerts.

### Interview answer:

> "I would first identify the filesystem and whether the problem is disk or inode exhaustion. I would determine what is consuming the space and avoid manually deleting Blob Store files. If necessary, I would increase storage capacity, then use appropriate cleanup policies and verify Nexus functionality. Finally, I would add monitoring and alerting to prevent recurrence."

---

## 14. USERS SUDDENLY GET 401 ERRORS AFTER A CERTIFICATE CHANGE

**Q: After a certificate or authentication change, users suddenly cannot log in to Nexus and receive 401 errors. What would you check?**

### Answer:

401 means authentication is failing.

I would check:

1. Was there a recent certificate/authentication change?

2. If LDAP/AD is being used:
   - Is LDAP reachable?
   - Are bind credentials valid?
   - Is the LDAP certificate trusted?
   - Is the LDAP connection working?

3. If SSO is used:
   - Is the identity provider available?
   - Is the certificate valid?
   - Are client/issuer settings correct?

4. Check Nexus security logs.

5. Test with a known valid account.

6. Verify that the issue affects:
   - All users
   - One user
   - CI/CD accounts
   - Docker users
   - Maven users

This helps determine whether the issue is global or user-specific.

### Interview answer:

> "Since the error is 401, I would focus on authentication rather than repository permissions. I would check the recent certificate or identity-provider change, LDAP/SSO connectivity, trust configuration, credentials, and Nexus security logs. I would also test with a known working account to determine the scope."

---

## 15. NEXUS USERS ARE GETTING 403 ERRORS

**Q: Users can log in successfully, but they receive 403 Forbidden when trying to download or upload artifacts. What does this mean?**

### Answer:

401 and 403 are different.

**401:**

    Authentication problem.

**403:**

    User is authenticated but does not have permission.

So I would investigate authorization.

Check:

    User
      |
      v
    Role
      |
      v
    Privileges
      |
      v
    Repository

For download:

    Browse + Read

For upload:

    Add

For modification:

    Edit

For deletion:

    Delete

Also check Content Selectors.

### Example:

    User has Read permission
             |
             v
    Content Selector allows:
        project-a/*
             |
             v
    User requests:
        project-b/*
             |
             v
          403

### Interview answer:

> "A 403 tells me that authentication is probably successful but authorization is failing. I would check the user's roles, privileges, repository permissions, and content selectors. I would verify that the required privilege exists for the specific operation."

---

## 16. A NEXUS PROXY REPOSITORY IS EXTREMELY SLOW

**Q: Developers complain that dependencies take several minutes to download through a Nexus proxy. How would you troubleshoot?**

### Answer:

I would identify whether the delay is:

    Developer -> Nexus

    OR

    Nexus -> Upstream

Check:

1. Nexus response time.

2. Cache status.

If the artifact is already cached:

    Developer
      |
      v
    Nexus Cache
      |
      v
    Fast response

If it is not cached:

    Developer
      |
      v
    Nexus
      |
      v
    Internet
      |
      v
    Upstream
      |
      v
    Slow response

3. Check upstream repository latency.

4. Check corporate proxy.

5. Check DNS.

6. Check network latency.

7. Check CPU, memory, and disk I/O.

8. Check Blob Store performance.

9. Check Nexus logs.

### Interview answer:

> "I would determine whether the delay is between the client and Nexus or between Nexus and the upstream repository. I would check cache hits, upstream latency, corporate proxy, DNS, network, system resources, Blob Store performance, and Nexus logs."

---

## 17. CI/CD PIPELINES FAIL BECAUSE NEXUS IS UNAVAILABLE

**Q: Your Jenkins pipelines depend on Nexus. Nexus goes down and hundreds of production builds fail. How would you handle this?**

### Answer:

First, I would treat Nexus as a critical dependency.

### Immediate response:

1. Confirm Nexus availability.
2. Identify the scope of impact.
3. Check whether existing artifacts are still available.
4. Start Nexus recovery.
5. Communicate the production impact.

Then investigate the root cause.

### Important consideration:

If dependencies are already cached locally on CI runners, some builds may still work.

But new dependencies may fail.

After Nexus is restored:

1. Test authentication.
2. Test artifact download.
3. Test artifact upload.
4. Run a controlled pipeline.
5. Monitor Nexus.

### Long-term improvement:

- Proper backup
- Monitoring
- Alerting
- Disaster recovery
- Highly available architecture where appropriate
- Documented incident procedure

### Interview answer:

> "I would first confirm the outage and identify the affected pipelines. I would recover Nexus according to the production runbook, then validate download and upload operations before resuming large-scale builds. After recovery, I would perform a root-cause analysis and improve monitoring, backup, and disaster recovery."

---

## 18. A SNAPSHOT REPOSITORY HAS GROWN TO HUNDREDS OF GB

**Q: Your Maven snapshot repository has grown massively and is consuming most of the storage. What would you do?**

### Answer:

SNAPSHOT repositories can grow quickly because CI pipelines may produce many versions.

### Example:

    1.0-SNAPSHOT
    1.0-SNAPSHOT
    1.0-SNAPSHOT
    ...
    thousands of builds

I would:

1. Identify how much storage snapshots consume.

2. Determine the organization's retention requirement.

3. Create a cleanup policy.

### Example:

    Delete snapshots older than 30 days.

Or based on:

- Age
- Number of versions
- Usage/retention requirements

4. Run cleanup.

5. Monitor storage.

6. Prevent the problem from returning by implementing appropriate retention policies and alerts.

### Important:

Do not blindly delete snapshots that are still required by active development or release processes.

### Interview answer:

> "Snapshot repositories are expected to grow quickly. I would first determine the retention requirement, then configure a cleanup policy based on age or retention rules. I would run the cleanup process, monitor storage, and configure alerts so the repository does not reach a critical capacity."

---

## 19. A PACKAGE EXISTS IN NEXUS BUT MAVEN STILL GETS 404

**Q: You can see the artifact in the Nexus UI, but Maven gets a 404 Not Found. What would you check?**

### Answer:

This is a very common production troubleshooting scenario.

First verify that the artifact exists in the SAME repository that Maven is accessing.

### Example:

Nexus UI:

    Artifact exists in:
        maven-releases

Maven:

    Requesting:
        maven-snapshots

This can cause a 404.

Check:

1. Repository URL in settings.xml.

2. Group repository membership.

3. Repository order.

4. Artifact coordinates:

    Group ID
    Artifact ID
    Version

5. Release vs Snapshot.

### Example:

    1.2.0-SNAPSHOT
        |
        v
    Snapshot Repository

    1.2.0
        |
        v
    Release Repository

6. Content selector permissions.

7. Repository routing rules if a proxy is involved.

8. Nexus logs.

### Interview answer:

> "I would first confirm that Maven is requesting the same repository where the artifact exists. Then I would verify the repository URL, group membership and order, artifact coordinates, release versus snapshot configuration, content selectors, and routing rules. Finally, I would check Nexus logs."

---

## 20. NEXUS BACKUP EXISTS, BUT YOU NEED TO RESTORE AFTER A FAILURE

**Q: Your production Nexus server is corrupted and you need to restore it. How would you approach the recovery?**

### Answer:

I would follow the organization's documented disaster recovery procedure rather than improvising.

First identify:

1. What failed?
2. Is the server recoverable?
3. Is the Blob Store intact?
4. Is the configuration intact?
5. What backup is available?

Important recovery data can include:

- Nexus configuration
- Repository configuration
- Security configuration
- Blob Store/artifact data
- Supporting infrastructure configuration

### Recovery process conceptually:

    Backup
      |
      v
    Restore infrastructure
      |
      v
    Restore Nexus configuration/data
      |
      v
    Verify Blob Store
      |
      v
    Start Nexus
      |
      v
    Health checks
      |
      v
    Test authentication
      |
      v
    Test artifact download
      |
      v
    Test artifact upload
      |
      v
    Test CI/CD
      |
      v
    Production recovery

After recovery, verify:

- Users can log in.
- Repositories are available.
- Artifacts can be downloaded.
- Artifacts can be uploaded.
- Proxy repositories can access upstream sources.
- CI/CD pipelines work.
- Storage is healthy.

### Interview answer:

> "I would follow the documented disaster recovery procedure. I would first determine what data is intact and which backup is required. I would restore the infrastructure, Nexus configuration, and required artifact storage according to the supported recovery process. After startup, I would validate health, authentication, repositories, artifact download/upload, proxy connectivity, and CI/CD before declaring the service recovered."

---

## IMPORTANT PRODUCTION TROUBLESHOOTING MINDSET

When you get a production Nexus problem, DON'T immediately say:

"Nexus is broken."

Instead, think in layers:

    CLIENT
      |
      v
    NETWORK
      |
      v
    LOAD BALANCER / PROXY
      |
      v
    NEXUS APPLICATION
      |
      v
    REPOSITORY
      |
      v
    SECURITY
      |
      v
    BLOB STORE / STORAGE
      |
      v
    UPSTREAM REPOSITORY

For every incident ask:

1. What exactly is failing?
2. Who is affected?
3. When did it start?
4. What changed recently?
5. What HTTP error are we getting?
6. Is Nexus reachable?
7. Is authentication working?
8. Is authorization working?
9. Is the repository configured correctly?
10. Is storage healthy?
11. Is the upstream repository reachable?
12. What do the Nexus logs say?

---

## MOST IMPORTANT HTTP CODES

    401 = Authentication failed

    403 = Authentication succeeded but permission is denied

    404 = Resource/artifact not found

    408 = Request timeout

    429 = Too many requests/rate limiting

    500 = Nexus/server internal error

    502 = Bad gateway / upstream communication problem

    503 = Service unavailable

    504 = Gateway timeout

---

## BEST ONE-LINE INTERVIEW FORMULA

> "First I would identify the scope and exact error, then check client configuration, network connectivity, authentication, authorization, repository configuration, storage/upstream connectivity, and finally Nexus logs. After fixing the issue, I would validate the complete workflow and monitor the system to make sure the problem does not recur."
