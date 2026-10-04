# Maven `pom.xml` — Nexus Configuration and Explanation

**`pom.xml`** has `distributionManagement`, which tells `mvn deploy` where to upload. Release versions go to `maven-releases` and `-SNAPSHOT` versions go to `maven-snapshots`.

The **`<id>` values must match** between the two files (`nexus-releases` and `nexus-snapshots`). If they don't, you'll get a 401 error on deploy.

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.example</groupId>
  <artifactId>demo-app</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!-- Change to your Nexus URL -->
    <nexus.url>http://localhost:8081</nexus.url>
  </properties>
 
  <!-- Where "mvn deploy" uploads artifacts.
       The <id> values must match the <server> ids in settings.xml -->
  <distributionManagement>
    <repository>
      <id>nexus-releases</id>
      <name>Nexus Releases</name>
      <url>${nexus.url}/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
      <id>nexus-snapshots</id>
      <name>Nexus Snapshots</name>
      <url>${nexus.url}/repository/maven-snapshots/</url>
    </snapshotRepository>
  </distributionManagement>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>

```
## `pom.xml` explained, section by section

POM stands for **Project Object Model**. It is the file Maven reads to learn what your project is, how to build it, and where to publish it.

### 1. The XML header and root tag

<project xmlns="http://maven.apache.org/POM/4.0.0" ...>

- `<project>` is the root tag, and everything else sits inside it.
- The `xmlns` and `xsi:schemaLocation` lines point to Maven's schema (XSD). They let your IDE validate the file and autocomplete tags. Maven works without them, but you should keep them.

### 2. `<modelVersion>4.0.0</modelVersion>`

This is the version of the POM format itself, not your project's version. It has been `4.0.0` for Maven 2, 3 and 4, so you never change it.

### 3. Coordinates (GAV)

<groupId>com.example</groupId>
<artifactId>demo-app</artifactId>
<version>1.0.0-SNAPSHOT</version>
<packaging>jar</packaging>

These three tags are the project's unique address, called **GAV** (Group, Artifact, Version). Nexus stores your file using this address.

| Tag | Meaning | Example |
|---|---|---|
| `groupId` | Your organization or company, usually a reversed domain | `com.example` |
| `artifactId` | The project name | `demo-app` |
| `version` | The release number | `1.0.0-SNAPSHOT` |

In Nexus, this project ends up at:

`com/example/demo-app/1.0.0-SNAPSHOT/demo-app-1.0.0-SNAPSHOT.jar`

**`-SNAPSHOT` matters a lot.** It means "work in progress". Maven decides where to deploy based on this suffix:

- Version ends with `-SNAPSHOT`: it goes to the **snapshot repository**. You can redeploy it again and again.
- Version has no suffix, like `1.0.0`: it goes to the **release repository**. It is immutable, so you cannot overwrite it (Nexus blocks redeploy by default).

**`<packaging>`** is the type of output file. `jar` is the default, so you can omit it. Other values are `war`, `pom` (for parent or multi-module projects) and `ear`.

### 4. `<properties>`

<properties>
  <maven.compiler.source>17</maven.compiler.source>
  <maven.compiler.target>17</maven.compiler.target>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <nexus.url>http://localhost:8081</nexus.url>
</properties>

Properties are variables. You define them once and reuse them with `${name}`.

- `maven.compiler.source` and `maven.compiler.target` tell the compiler which Java version to use. Here that is Java 17.
- `project.build.sourceEncoding` makes the build use UTF-8, so builds behave the same on every machine.
- `nexus.url` is a custom property that we created. We use it later as `${nexus.url}`, so if your Nexus URL changes, you edit it in one place.

### 5. `<distributionManagement>` (the Nexus part)

<distributionManagement>
  <repository>
    <id>nexus-releases</id>
    <name>Nexus Releases</name>
    <url>${nexus.url}/repository/maven-releases/</url>
  </repository>
  <snapshotRepository>
    <id>nexus-snapshots</id>
    <name>Nexus Snapshots</name>
    <url>${nexus.url}/repository/maven-snapshots/</url>
  </snapshotRepository>
</distributionManagement>

This section is used **only when you run `mvn deploy`**. It answers the question "where should I upload the built artifact?"

- `<repository>` is the target for release versions.
- `<snapshotRepository>` is the target for `-SNAPSHOT` versions.
- `<url>` points to a repository you created in Nexus. The pattern is `http://<host>:8081/repository/<repo-name>/`.
- `<name>` is just a label for humans.

**`<id>` is the most important tag here.** Maven does not store passwords in the POM. When it deploys, it takes the `<id>` (for example `nexus-releases`) and looks for a `<server>` with the same id in `settings.xml` to get the username and password. This is why I told you the ids must match. If they differ, you get `401 Unauthorized`.

### 6. `<dependencies>`

<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
  </dependency>
</dependencies>

These are the libraries your project needs. Each one is identified by its own GAV. Maven downloads them for you. With the `settings.xml` mirror, it downloads them **from Nexus** (`maven-public`) instead of directly from the internet.

**`<scope>`** controls when the library is available:

| Scope | Available during | Packaged in final app? |
|---|---|---|
| `compile` (default) | compile, test, run | Yes |
| `provided` | compile, test | No (the server supplies it) |
| `runtime` | test, run | Yes |
| `test` | test only | No |

JUnit is `test` because only your test code needs it. It should not ship in your jar.

### How the build flows

1. You run `mvn clean deploy`.
2. Maven reads **dependencies** and downloads them, through the mirror in `settings.xml`, from Nexus.
3. It compiles with Java 17, runs tests, and packages a `jar`.
4. It checks `<version>`. If it ends in `-SNAPSHOT`, it uses `<snapshotRepository>`. Otherwise it uses `<repository>`.
5. It takes the `<id>`, finds the matching `<server>` in `settings.xml`, and uses that username and password.
6. It uploads the jar to Nexus.

### Common mistakes

- **401 Unauthorized**: the `<id>` in the POM and the `<server>` id in `settings.xml` don't match, or the password is wrong.
- **400 Bad Request or "repository does not allow updating assets"**: you tried to redeploy a release version. Bump the version, or change the repo's deployment policy in Nexus.
- **Snapshot going to the releases repo, or the reverse**: check that your `<version>` suffix is correct.
- **Wrong repository name in the URL**: the repo name must exactly match what you created in Nexus.
