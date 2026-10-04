# Maven `settings.xml` — Servers + Mirrors

## Complete File

```

<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <!-- Credentials: ids must match <id> in pom.xml distributionManagement -->
  <servers>
    <server>
      <id>nexus-releases</id>
      <username>deploy-user</username>
      <password>your-password</password>
    </server>
    <server>
      <id>nexus-snapshots</id>
      <username>deploy-user</username>
      <password>your-password</password>
    </server>
    <!-- Used by the mirror below -->
    <server>
      <id>nexus-public</id>
      <username>deploy-user</username>
      <password>your-password</password>
    </server>
  </servers>

  <!-- Route ALL dependency downloads through Nexus (maven-public group repo) -->
  <mirrors>
    <mirror>
      <id>nexus-public</id>
      <name>Nexus Public Group</name>
      <url>http://localhost:8081/repository/maven-public/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>

  <profiles>
    <profile>
      <id>nexus</id>
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>

  <activeProfiles>
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
</settings>

```

## 1. `<servers>` — Credentials

The `<servers>` section stores the username and password Maven uses when connecting to Nexus repositories.

    <servers>

      <server>
        <id>nexus-releases</id>
        <username>deploy-user</username>
        <password>your-password</password>
      </server>

      <server>
        <id>nexus-snapshots</id>
        <username>deploy-user</username>
        <password>your-password</password>
      </server>

      <server>
        <id>nexus-public</id>
        <username>deploy-user</username>
        <password>your-password</password>
      </server>

    </servers>

### Server IDs

| Server ID | Used for |
|---|---|
| `nexus-releases` | Uploading release versions |
| `nexus-snapshots` | Uploading SNAPSHOT versions |
| `nexus-public` | Downloading dependencies through Nexus |

### Important

The `<id>` in `settings.xml` must match the `<id>` used by Maven.

    pom.xml
        |
        |-- nexus-releases
        |
        v
    settings.xml
        |
        |-- nexus-releases
        |-- username
        |-- password

If the IDs do not match:

    401 Unauthorized

The `<server>` contains credentials.

It does NOT contain the repository URL.

The repository URL is defined in the `pom.xml` or in the mirror configuration.

---

## 2. `<mirrors>` — Redirect Downloads

The `<mirrors>` section tells Maven to redirect dependency downloads through Nexus.

    <mirrors>

      <mirror>
        <id>nexus-public</id>
        <name>Nexus Public Group</name>
        <url>http://localhost:8081/repository/maven-public/</url>
        <mirrorOf>*</mirrorOf>
      </mirror>

    </mirrors>

### What each tag means

- `<id>nexus-public</id>` connects the mirror to the `nexus-public` server credentials.
- `<url>` points to the Nexus `maven-public` group repository.
- `<mirrorOf>*</mirrorOf>` means Maven sends all repository download requests through Nexus.

### Download flow

    Maven
      |
      v
    settings.xml
      |
      v
    <mirrors>
      |
      |-- mirrorOf = *
      |
      v
    Nexus maven-public
      |
      +----> Hosted repositories
      |
      +----> Proxy repositories
      |
      +----> Maven Central
      |
      v
    <servers>
      |
      |-- nexus-public
      |-- username
      |-- password

### Why use a mirror?

- Nexus caches dependencies.
- Builds can be faster.
- The organization controls dependency downloads.
- Maven Central does not need to be accessed directly by every build.
- All dependency downloads can go through one Nexus URL.

### Common `mirrorOf` values

| Value | Meaning |
|---|---|
| `*` | Everything |
| `central` | Only Maven Central |
| `external:*` | Everything except localhost and file-based repositories |
| `*,!my-repo` | Everything except `my-repo` |

---

# Complete `settings.xml`

    <?xml version="1.0" encoding="UTF-8"?>

    <settings
        xmlns="http://maven.apache.org/SETTINGS/1.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                            https://maven.apache.org/xsd/settings-1.0.0.xsd">

      <servers>

        <server>
          <id>nexus-releases</id>
          <username>deploy-user</username>
          <password>your-password</password>
        </server>

        <server>
          <id>nexus-snapshots</id>
          <username>deploy-user</username>
          <password>your-password</password>
        </server>

        <server>
          <id>nexus-public</id>
          <username>deploy-user</username>
          <password>your-password</password>
        </server>

      </servers>


      <mirrors>

        <mirror>
          <id>nexus-public</id>
          <name>Nexus Public Group</name>
          <url>http://localhost:8081/repository/maven-public/</url>
          <mirrorOf>*</mirrorOf>
        </mirror>

      </mirrors>


      <profiles>

        <profile>

          <id>nexus</id>

          <repositories>

            <repository>
              <id>central</id>
              <url>http://central</url>

              <releases>
                <enabled>true</enabled>
              </releases>

              <snapshots>
                <enabled>true</enabled>
              </snapshots>

            </repository>

          </repositories>


          <pluginRepositories>

            <pluginRepository>
              <id>central</id>
              <url>http://central</url>

              <releases>
                <enabled>true</enabled>
              </releases>

              <snapshots>
                <enabled>true</enabled>
              </snapshots>

            </pluginRepository>

          </pluginRepositories>

        </profile>

      </profiles>


      <activeProfiles>

        <activeProfile>nexus</activeProfile>

      </activeProfiles>

    </settings>

---

# How `pom.xml` and `settings.xml` Work Together

    mvn clean deploy
            |
            +-----------------------------+
            |                             |
            v                             v
       DOWNLOAD                        UPLOAD
      DEPENDENCIES                   ARTIFACT
            |                             |
            v                             v
    pom.xml <dependencies>       pom.xml <distributionManagement>
            |                             |
            v                             v
    settings.xml <profiles>      settings.xml <servers>
            |                             |
            v                             v
    settings.xml <mirrors>       Matching server ID
            |                             |
            v                             v
    Nexus maven-public            Nexus credentials
            |                             |
            +-------------+---------------+
                          |
                          v
                        Nexus

---

# Download Flow

    pom.xml
       |
       |-- <dependencies>
       |
       v
    settings.xml
       |
       |-- <profiles>
       |      |
       |      +-- <repositories>
       |
       v
    <mirrors>
       |
       |-- mirrorOf = *
       |
       v
    Nexus maven-public
       |
       v
    <servers>
       |
       |-- nexus-public
       |-- username
       |-- password
       |
       v
    Dependency downloaded

---

# Upload Flow

    mvn clean deploy
            |
            v
    pom.xml
            |
            |-- <distributionManagement>
            |
            +---- Release
            |       |
            |       v
            |   nexus-releases
            |       |
            |       v
            |   settings.xml
            |       |
            |       v
            |   <server>
            |       |
            |       +-- username
            |       +-- password
            |
            +---- SNAPSHOT
                    |
                    v
                nexus-snapshots
                    |
                    v
                settings.xml
                    |
                    v
                  <server>
                    |
                    +-- username
                    +-- password

---

# Release vs SNAPSHOT

    pom.xml version
           |
           +-----------------------------+
           |                             |
           v                             v
         1.0.0                    1.0.0-SNAPSHOT
           |                             |
           v                             v
    Release repository           Snapshot repository
           |                             |
           v                             v
    maven-releases               maven-snapshots
           |                             |
           v                             v
    nexus-releases               nexus-snapshots
           |                             |
           v                             v
      settings.xml                settings.xml
           |                             |
           v                             v
       credentials                  credentials

---

# Important ID Matching

    pom.xml

    <repository>
        <id>nexus-releases</id>
    </repository>

            |
            | MUST MATCH
            v

    settings.xml

    <server>
        <id>nexus-releases</id>
        <username>deploy-user</username>
        <password>your-password</password>
    </server>

For snapshots:

    pom.xml

    <snapshotRepository>
        <id>nexus-snapshots</id>
    </snapshotRepository>

            |
            | MUST MATCH
            v

    settings.xml

    <server>
        <id>nexus-snapshots</id>
        <username>deploy-user</username>
        <password>your-password</password>
    </server>

For dependency downloads:

    settings.xml

    <mirror>
        <id>nexus-public</id>
        <url>http://localhost:8081/repository/maven-public/</url>
    </mirror>

            |
            | MATCHES
            v

    <server>
        <id>nexus-public</id>
        <username>deploy-user</username>
        <password>your-password</password>
    </server>

---

# Common Mistakes

- **401 Unauthorized**: `<server>` ID doesn't match the POM ID, or the password is wrong.
- **Dependencies still come from the internet**: the mirror is missing, or `mirrorOf` doesn't cover the repository.
- **Settings seem ignored**: the file is in the wrong place. Check `~/.m2/settings.xml`.
- **Check effective settings**:

    mvn help:effective-settings

- **Committed to Git with a password**: keep `settings.xml` out of source control.
- **Mirror needs login but there is no `nexus-public` server**: dependency downloads can fail with `401` or `403`.
- **Snapshot going to releases repository**: check the `<version>` in `pom.xml`.
- **Release going to snapshots repository**: check the `<version>` in `pom.xml`.
- **Wrong repository name**: the repository name in the Nexus URL must exactly match the repository created in Nexus.

---

# Password Security

Do not commit real passwords:

    <password>my-real-password</password>

Maven can encrypt passwords.

Create the master password:

    mvn --encrypt-master-password yourMasterPass

Store the generated value in:

    ~/.m2/settings-security.xml

Encrypt the actual Nexus password:

    mvn --encrypt-password yourRealPassword

Then use the generated encrypted value:

    <password>{xyz123...}</password>

For CI/CD, another option is an environment variable:

    <password>${env.NEXUS_PASSWORD}</password>

---
