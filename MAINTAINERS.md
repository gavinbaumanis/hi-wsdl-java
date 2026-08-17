# Maintainer guide

Paths are relative to the repository root (directory containing **`pom.xml`**).

## Release lines

**Documentation convention:** README, CONTRIBUTING, CHANGELOG, and integrator-facing text use **version numbers only** - never Git branch names.

| Version | Java | APIs | `Service` stubs |
| ------- | ---- | ---- | ----------------- |
| **1.6.4** | 8 | **`javax.xml.ws`**, **`javax.xml.bind`** | **14** (standard HI B2B) |
| **1.6.6** | 11 | **Jakarta** XML WS / Bind | **14** (standard HI B2B) |
| **1.7.1** | 11 | **Jakarta** XML WS / Bind | **26** (full MCA) |

**Git branch mapping (maintainers / checkout only - do not use in integrator docs):**

| Version | Git branch |
| ------- | ---------- |
| **1.6.4** | `java-8-javax` |
| **1.6.6** | `java-11-jakarta` |
| **1.7.1** | `java-11-jakarta-full-wsdl` |

**This 1.6.4 line (`1.6.4-SNAPSHOT`):** Java **8**, committed **`javax`** generated types, **14** primary HI B2B **`@WebServiceClient`** services (no **`wsimport`** / XJC in this POM). **`hi-b2b-client`** **1.6.4** uses **`hi-wsdl`** at the **same version** (**`${project.version}`**) - **`mvn install`** here before an unpublished client **`verify`**. GA **1.6.4** pairs ship to Maven Central together. Type regeneration for new HI releases is on **`1.6.6`** / **`1.7.1`** lines.

## Artifact

- **`au.gov.nehta:hi-wsdl`** - HI WSDL on the classpath + pre-generated JAX-WS/JAXB types.
- **Not included:** facade clients, TLS/signing, or runtime filesystem WSDL resolution (**`HiWsdlArtifactRoot`** lives in **hi-b2b-client-java**).

## Build (`1.6.4` line)

- **`maven.compiler.release` 8**
- Compile deps: **`jaxb-api` 2.3.1**, **`jaxws-api` 2.3.1** - no **`jaxws-rt`** in this POM
- **`maven-enforcer-plugin`:** bans Metro **`webservices-rt`** / **`webservices-api`** / **`webservices-extra*`** / **`metro-*`** and all **`jakarta.xml.*`** / **`jakarta.jws`** API coordinates ( **`javax`** compile deps only)
- Consumers (e.g. **hi-b2b-client**): **`com.sun.xml.ws:jaxws-rt` 2.3.7** (Eclipse EE4J, last **2.3.x** on Central for Java **8**)
- **`maven-gpg-plugin`:** skipped unless **`-Dgpg.skip=false`**
- **`maven-javadoc-plugin`:** **`doclint=none`**, **`verbose=false`**, **`quiet=true`**, **`failOnWarnings=false`**, **`detectOfflineLinks=false`**, **`source=${maven.compiler.release}`** (8).
- **No source regeneration** on **`1.6.4`** - no licensed MCA **`wsdls/xml`** tree and no **`-Pregenerate-sources`** profile. **`src/main/java`** and **`hi_override/`** are committed as-is.

## Release

Publishing uses **`central-publishing-maven-plugin`** (Sonatype Central Portal). Copy **`settings.xml.example`** -> **`settings.xml`**, server id **`central`**.

**Parallel release lines (maintainers only):** each Git branch publishes a **different Maven version** - integrators choose by coordinate, not branch name.

| Branch | Java | HI client / WSDL version | Facades |
| ------ | ---- | ------------------------ | ------- |
| **`java-8-javax`** | 8 / javax | **1.6.4** | 14 |
| **`java-11-jakarta`** | 11 / Jakarta | **1.6.6** | 14 |
| **`java-11-jakarta-full-wsdl`** | 11 / Jakarta | **1.7.1** | 26 |

Release **`hi-wsdl`** and **`hi-b2b-client`** at the **same GA version** on the matching branch pair before integrators upgrade.

### SNAPSHOT or manual GA

1. Update **CHANGELOG.md** (and **`pom.xml`** / SCM **`<tag>`** for manual GA).
2. **`mvn -B "-Prelease" clean verify`**
3. **`mvn -B "-Prelease" deploy`**

Git/SCM settings for **`maven-release-plugin`** live in **`pom.xml`** properties (**`scm.repo.url`**, **`release.*`**). Tags default to **`{artifactId}-{version}`** (e.g. **`hi-wsdl-1.6.4`**).

### Automated GA (`maven-release-plugin`)

Run on the **target branch** with a **clean** working tree. The plugin commits version bumps, creates the release tag, deploys from the tag checkout, bumps to the next **`-SNAPSHOT`**, and **pushes branch + tag** (**`pushChanges`** / **`remoteTagging`** in **`pom.xml`**). Git remote credentials (SSH or HTTPS) must work non-interactively.

```text
mvn -B "-Prelease" release:prepare release:perform -DreleaseVersion=1.6.4 -DdevelopmentVersion=1.6.7-SNAPSHOT -Dtag=hi-wsdl-1.6.4
```

Replace versions and **`-Dtag`** for the line you are on. Cut **`hi-wsdl`** and **`hi-b2b-client`** at the **same** GA (this line: **`hi-wsdl-1.6.4`** and **`hi-b2b-client-1.6.4`**). Omit **`-D...`** only if you accept interactive prompts.

**After success:** confirm the artifact on Central; repeat on the paired types/client repo. No extra Git steps unless push failed (then **`git push origin <branch>`** and **`git push origin <tag>`**).

**`-Dgpg.skip=false`** is equivalent to **`-Prelease`** for signing.

## Copyright

Copyright 2012 NEHTA. Copyright 2021-2026 ADHA. Apache License 2.0 - see **LICENSE.txt**.
