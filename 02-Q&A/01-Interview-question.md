### Basics

**1. What is Nexus Repository Manager?**

It's a tool from Sonatype that works like a **storage room for software files**. Developers keep their libraries, packages, and build outputs (artifacts) there and share them from one place.

**2. What are the main editions?**

Two: a **free edition** (OSS, called Community Edition in newer versions, which has usage limits) and a **paid Pro edition** with extra features like high availability, staging, and security scanning.

**3. What are the main features?**

One central place for artifacts, support for many formats, connection with CI/CD tools like Jenkins, access control by roles, and (in Pro) high availability.

**4. Which formats does it support?**

Maven, npm, Docker, PyPI, NuGet, RubyGems, Helm, Go, and more. Anything else can be stored in a **Raw** repository.

**5. What is a Proxy repository?**

It sits in front of a public site like Maven Central. The first download is fetched from the internet and **saved (cached)**. After that, everyone gets the saved copy, so it's faster and works even if the public site is down.

**6. What is a Hosted repository?**

A **private storage space for your own company's files**, such as internal libraries.

**7. What is a Group repository?**

It **combines several repositories under one URL**, so developers configure just one address.

**8. How do you open the Nexus interface?**

In a browser, go to `http://<server-ip>:8081`.

**9. What are the default credentials?**

⚠️ **Update:** `admin / admin123` was only for **old versions**. In current versions, the username is `admin`, and the **first-time password is randomly generated** and saved in a file called `admin.password` inside the Nexus data folder (`sonatype-work/nexus3` or `nexus-data`). Nexus asks you to change it at first login.

**10. How do you change the admin password?**

Log in, open the **Security → Users** section (under Administration), click the `admin` user, and change the password.

---

### Installation and Setup

**11. What are the system requirements?**

⚠️ **Update:** The numbers in your document (1 GB RAM, Java 8/11) are old. In practice, you need a supported **Java version** (it depends on the Nexus version), several CPU cores, **at least 4-8 GB RAM** for real use, and plenty of disk space. Say: "It depends on the version; I check Sonatype's system requirements page."

**12. How do you install Nexus on Linux?**

Install Java, download and extract the `tar.gz` file, create a dedicated `nexus` user, give it ownership of the folders, set that user to run Nexus, create a **systemd service**, and start it.

**13. How do you install Nexus on Windows?**

Download the Nexus bundle from Sonatype, extract it, and run it (or install it as a Windows service). It then starts on port 8081.

**14. What is the `nexus.vmoptions` file for?**

It holds the **Java (JVM) settings**, mainly memory settings like heap size (`-Xms`, `-Xmx`).

**15. How do you set up SSL?**

Two ways: put a **reverse proxy (Nginx/Apache)** in front, install the certificate there, and forward requests to Nexus; or configure Nexus's own server with a **keystore**. The reverse proxy is more common.

---

### Configuration and Security

**16. How do you create a role?**

Go to **Security → Roles → Create role**, give it a name, pick the permissions (privileges), and save.

**17. How do you assign a role to a user?**

Go to **Security → Users**, open the user, add the role, and save.

**18. How do you connect Nexus to LDAP?**

Go to **Security → LDAP → Create connection**, enter the server details, search base, and a bind account, test the connection, and save. Then map LDAP groups to Nexus roles and enable the LDAP realm.

**19. What is a Blob Store?**

It's the **place where the actual artifact files are stored** (on disk or in cloud storage). The database only keeps information about them.

**20. How do you create a Blob Store?**

Go to **Repository → Blob Stores → Create blob store**, choose the type (File or S3), set the name and path, and save.

---

### Usage and Management

**21. How do you create a Maven hosted repository?**

Go to **Repositories → Create repository → maven2 (hosted)**, choose the version policy (Release or Snapshot), and save.

**22. How do you make Maven use Nexus?**

In `settings.xml`, add a **mirror** that points to the Nexus group URL, and add your login under `<servers>`.

**23. How do you upload an artifact?**

Use the **Upload** option in the web interface, or use `mvn deploy` or the REST API for automation.

**24. How do you download an artifact?**

Open **Browse**, find the file, and click the download link. Build tools download automatically.

**25. How do you set up a cleanup policy?**

Go to **Repository → Cleanup Policies**, create a policy (for example, "older than 30 days"), then **assign it to a repository**. Remember to run **Compact blob store** to free the disk space.

---

### CI/CD Integration

**26. How do you connect Jenkins with Nexus?**

Install a Nexus plugin (Nexus Artifact Uploader or the Sonatype Nexus Platform plugin), store the Nexus login in Jenkins credentials, and add an upload step in the pipeline.

**27. How do you use Nexus for Docker images?**

Create a **Docker hosted repository** with its own port, then use `docker login`, `docker tag`, `docker push`, and `docker pull` with the Nexus address. HTTPS is needed.

**28. What is `nexus-staging-maven-plugin`?**

⚠️ **Update:** It was used to upload artifacts to a **staging repository** and release them, mostly for Maven Central publishing. It is **now deprecated** because Maven Central's publishing process changed. Say: "It was used for staging and releasing; today we use the newer Central publishing plugin or Nexus 3 promotion features."

**29. How do you deploy with `mvn deploy`?**

Add `<distributionManagement>` with the repository URLs in `pom.xml`, put your login in `settings.xml` (the server ID must match), and run `mvn deploy`.

**30. How do you upload to Nexus from a Jenkins pipeline?**

Add an upload step to the pipeline (the plugin step or `mvn deploy`), give it the Nexus URL, repository, artifact details, and credentials.

---

### Advanced Features

**31. What are staging repositories for?**

They are a **waiting area** where artifacts are tested and approved before going to the final release repository.

**32. How do you create a staging profile?**

⚠️ **Update:** Staging profiles belong to **older Nexus 2**. In Nexus 3, staging is a **Pro feature** that uses tags and moving components between repositories. Say: "In Nexus 3 we stage using separate repositories and promote the tested artifact."

**33. How do you promote from staging to release?**

After tests and approval pass, **move or copy the same artifact** to the release repository (UI, REST API, or a pipeline step). Never rebuild it.

**34. Snapshot vs Release?**

A **snapshot** is a work-in-progress version (like `1.0-SNAPSHOT`) that can change. A **release** is the final, tested version that never changes.

**35. How do you handle large binary files?**

Store them in a **separate blob store**, set a size alert (soft quota), clean up old files regularly, and use fast disks.

---

### Backup and Restore

**36. How do you back up Nexus?**

⚠️ **Update:** Stopping Nexus and copying the data folder works, but the better way is to run the scheduled **database export (backup) task** while Nexus is running, then also **copy the blob stores**.

**37. How do you restore?**

Stop Nexus, restore the database backup and the blob store files, then start Nexus. If they don't match, run the **reconcile** task.

**38. Why back up regularly?**

To recover from **disk failure, corruption, or accidental deletion** without losing your artifacts.

**39. What goes in the backup?**

The **database**, the **blob stores**, and the **configuration files**.

**40. How do you schedule backups?**

Use Nexus's scheduled backup task for the database, and a **cron job** (or backup tool) for the blob stores. Also copy backups to another location.

---

### Best Practices

**41. Security best practices?**

Strong passwords, **HTTPS**, regular updates, **role-based access**, disabling anonymous access if not needed, and checking the **audit logs**.

**42. How do you improve performance?**

Give enough **memory and CPU**, use fast disks, **clean up old artifacts**, compact blob stores, and spread large workloads across blob stores.

**43. Benefits of a Group repository?**

**One URL** for developers, easier setup, fewer mistakes, and you can add or change repositories without touching developer machines.

**44. How do you handle large deployments?**

Use **high availability (Pro)** with multiple nodes, a load balancer, a shared database and blob store, good monitoring, and a tested backup plan.

**45. Why watch the logs?**

To **find problems early**, troubleshoot errors, and spot suspicious activity.

---

### Troubleshooting

**46. What if Nexus runs out of memory?**

Increase the **heap size** in `nexus.vmoptions`, restart, and monitor. Also check for heavy tasks or too many old artifacts.

**47. How do you fix slow performance?**

Check **CPU, memory, disk, and network**, review the logs, clean up old artifacts, run compact tasks, and avoid running heavy tasks at busy times.

**48. What causes access problems?**

Wrong **permissions or roles**, an expired password, **network/firewall** issues, a wrong **LDAP** setup, or an expired SSL certificate.

**49. How do you fix repository index problems?**

⚠️ **Update:** Indexes are an older (Nexus 2) idea. In Nexus 3, run the **"Rebuild repository search"** task, and for Maven run **"Rebuild Maven repository metadata"**.

**50. How do you fix a startup failure?**

Read the **logs** (`nexus.log`), check that **Java is installed** and is a supported version, check **file permissions** and the user running Nexus, check the **disk space**, and check that the **port isn't already in use**.
