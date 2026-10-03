### 1. Basics

**Q1. What is Nexus Repository Manager and why do we use it?**

It is a central place to store and manage build artifacts (JARs, Docker images, npm packages, etc.). Think of it as a private warehouse for everything your builds produce or download.

- It speeds up builds through caching.
- It gives control over which dependencies are allowed.
- It removes dependence on the internet and public repos.

*Cross: Why not just use Git for artifacts?*

Git is for source code. Binaries are large and change often, so they bloat Git. Nexus handles versions, metadata, and proxying.

---

**Q2. What are the types of repositories?**

- **Hosted:** you upload your own artifacts (e.g., `maven-releases`, `maven-snapshots`).
- **Proxy:** a cache of a remote repo like Maven Central. Nexus downloads on first request, then serves from its cache.
- **Group:** combines several repos behind one URL, so developers configure only one URL.

*Cross: In a group, which repo is searched first?*

The order in the member list. Put hosted first, then proxy, so your own artifacts win.

---

**Q3. Release vs Snapshot repository?**

- **Release:** immutable. Once you upload `1.0.0`, you can't overwrite it (the deployment policy is "Disable redeploy").
- **Snapshot:** for work in progress (`1.0-SNAPSHOT`). It can be overwritten, and old snapshots are cleaned up automatically.

*Cross: Why is immutability important?*

It guarantees the same version always means the same code, which gives reproducible builds.

---

**Q4. Nexus 2 vs Nexus 3?**

Nexus 3 supports many formats (Docker, npm, PyPI, NuGet, Helm, etc.), while Nexus 2 mainly supports Maven. Nexus 3 uses blob stores, has a REST API and scripting, and supports cleanup policies.
