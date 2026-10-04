# Concept Easy Explanation

### 1. Repository

A place where your software packages are stored and managed. Think of it as a **warehouse for packages**.

### 2. Hosted Repository

Your company's own private package storage. You **upload and store your own artifacts** here.

### 3. Proxy Repository

A repository that gets packages from an external source such as Maven Central, npm, PyPI, etc. Nexus **downloads and caches** them for you.

### 4. Group Repository

Combines multiple repositories into **one URL**. Developers use one URL instead of knowing where every package comes from.

### 5. Blob Store

The actual storage location where Nexus keeps the package files. Think of it as the **hard drive/storage area** behind repositories.

### 6. Components & Assets

A **component** is a package/version, while **assets** are the actual files belonging to it. For example, one Maven component can contain a JAR and POM file.

### 7. Repository Format

Defines the package technology Nexus understands, such as **Maven, npm, Docker, NuGet, PyPI, Helm**, etc.

### 8. Cleanup Policies

Automatically remove old or unused packages to **save storage space**. For example, delete snapshots older than 30 days.

### 9. Security & RBAC

Controls **who can access what**. Users, roles, privileges, and permissions determine whether someone can read, upload, or delete artifacts.

### 10. Repository Manager / Artifact Management

Nexus acts as a central place to **store, download, cache, and distribute dependencies** used by your applications and CI/CD pipelines.

### 11. Repositories → Storage

Every repository needs storage. Nexus uses a **Blob Store** to physically save artifacts.

### 12. Repository URL

Each repository has a URL that developers, Maven, npm, Docker, or CI/CD tools use to **upload/download packages**.

### 13. Content Selectors

Rules that control **which specific artifacts** a user can access. Example: allow a team to access only `com/company/projectA/*`.

### 14. Privileges

Define **what action** a user can perform—read, browse, add, edit, delete, etc.

### 15. Roles

Roles group multiple privileges together. Instead of giving 10 permissions individually, create a role and assign it to a user.

### 16. Users

Accounts that authenticate to Nexus. Users receive permissions through their assigned **roles**.

### 17. Cleanup Policy

Defines which old artifacts Nexus should remove. Useful for controlling disk usage.

### 18. Routing Rules

Control which requests a **proxy repository is allowed to send upstream**. This can prevent unnecessary downloads from external repositories.

### 19. Repository Health / Status

Helps determine whether a repository is working correctly and whether Nexus can communicate with external repositories.

### 20. CI/CD Integration

Nexus can be integrated with tools such as **Jenkins, GitLab CI, GitHub Actions, Maven, npm, Docker**, etc., to store and retrieve build artifacts.

### 21. Maven Repository

Used to store and retrieve Java/Maven packages such as `.jar` and `.pom` files.

### 22. npm Repository

Used to store JavaScript/Node.js packages such as npm modules.

### 23. Docker Repository

Used to store and manage Docker container images.

### 24. Raw Repository

Used to store general files that don't belong to a specific package format.

### 25. Release vs Snapshot

Releases are stable versions; snapshots are development versions that can change frequently.

### 26. Versioning

Packages have versions such as `1.0.0`, `1.1.0`, and `2.0.0`, allowing different releases to coexist.

### 27. Repository Policy

Determines whether a repository accepts **release versions, snapshot versions, or both**.

### 28. Deployment Policy

Controls whether an artifact can be uploaded once, uploaded again, or overwritten.

### 29. Proxy Cache

Nexus keeps downloaded external packages locally so future requests can be faster and less dependent on the internet.

### 30. Negative Cache

Nexus remembers that a requested package was **not found upstream**, preventing repeated unnecessary requests.

### 31. Repository Routing

Controls which requests a proxy repository is allowed to send to an external repository.

### 32. Group Repository Order

Determines the order in which Nexus searches member repositories for an artifact.

### 33. Repository Connector

Defines how clients communicate with a repository, usually through HTTP/HTTPS.

### 34. HTTP vs HTTPS

HTTPS encrypts communication between the client and Nexus and should normally be preferred.

### 35. Anonymous Access

Allows users to access selected Nexus repositories without logging in.

### 36. LDAP Integration

Allows Nexus to authenticate users against an organization's LDAP/Active Directory system.

### 37. SSO / External Authentication

Allows users to authenticate through an external identity provider rather than maintaining separate Nexus passwords.

### 38. Audit / Security Logs

Records important activities such as logins, permission changes, and repository operations.

### 39. Health Checks

Helps determine whether Nexus and its repositories are functioning correctly.

### 40. Backup & Recovery

Protects Nexus configuration and repository data so the system can be restored after failure.

### 41. Maven `settings.xml`

Configuration file that tells Maven which Nexus repository to use and how to authenticate.

### 42. Maven `pom.xml`

Defines your project's dependencies, plugins, version, and other Maven settings.

### 43. Maven `distributionManagement`

Defines where Maven should upload your application's release and snapshot artifacts.

### 44. Maven Mirror

Allows Maven to send dependency requests through Nexus instead of directly accessing public repositories.

### 45. npm `.npmrc`

Configuration file that tells npm which Nexus registry to use and how to authenticate.

### 46. Docker Registry

A Nexus repository that stores Docker images and allows Docker clients to push/pull them.

### 47. Docker Push

Uploads a Docker image from your machine or CI/CD pipeline to Nexus.

### 48. Docker Pull

Downloads a Docker image from Nexus for running or deploying it.

### 49. Docker Login

Authenticates Docker with a Nexus Docker repository before push/pull operations.

### 50. Repository Authentication

Verifies that a user or tool is allowed to access Nexus.

### 51. SSL/TLS Certificate

Secures HTTPS communication between clients and Nexus.

### 52. API

Allows scripts and tools to interact with Nexus programmatically instead of using the UI.

### 53. REST API

HTTP-based API used to perform Nexus operations such as querying repositories and components.

### 54. NXRM Data Directory

Location where Nexus stores important application data and configuration.

### 55. Blob Store Management

Managing storage used by Nexus, including monitoring capacity and cleanup.

### 56. Repository Metadata

Information about packages, such as versions, names, checksums, and dependencies.

### 57. Checksum

A value used to verify that an artifact has not been corrupted or unexpectedly changed.

### 58. Artifact Coordinates

Identifies a package using information such as group, artifact name, and version.

### 59. Component Search

Allows you to find packages stored in Nexus by name, version, repository, etc.

### 60. Nexus Logs

Logs help troubleshoot errors involving repositories, authentication, storage, network, and downloads.

### 61. Repository Health Check

Checks whether a proxy repository can successfully communicate with its upstream repository.

### 62. Upstream Repository

The external repository from which a Nexus proxy downloads packages.

### 63. Remote URL

The external URL configured for a proxy repository, such as Maven Central.

### 64. HTTP Client Configuration

Controls how Nexus connects to external repositories, including timeouts and connection settings.

### 65. Proxy Authentication

Credentials Nexus uses when the upstream repository requires authentication.

### 66. Connection Timeout

Maximum time Nexus waits for a connection before considering it failed.

### 67. Request Timeout

Maximum time Nexus waits for a response from an external service.

### 68. Repository Cache

Local storage of packages downloaded from an upstream repository.

### 69. Cache Expiration

Determines when Nexus should check the upstream repository again for updated metadata.

### 70. Metadata

Information describing packages, versions, dependencies, and available artifacts.

### 71. Browse vs Read

Browse lets you see/search artifacts; Read lets you actually download them.

### 72. Add / Edit / Delete Privileges

Permissions controlling whether users can upload, modify, or remove artifacts.

### 73. Repository-Level Permissions

Permissions that apply to an entire repository.

### 74. Content Selector Permissions

Permissions restricted to specific artifact paths or patterns.

### 75. User Token / Access Token

Credentials that can be used by automation instead of a normal user password.

### 76. CI/CD Artifact Flow

Build tools create artifacts, Nexus stores them, and deployment tools retrieve them.

### 77. Artifact Promotion

Moving or copying an artifact from development/testing toward a release repository.

### 78. Immutable Artifacts

Released artifacts should generally not be changed or overwritten after publication.

### 79. Storage Monitoring

Watching disk/blob-store usage to prevent Nexus from running out of space.

### 80. High Availability / Disaster Recovery

Planning how Nexus remains available or is restored after infrastructure failure.
