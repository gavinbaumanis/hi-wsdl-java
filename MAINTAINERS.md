# Maintainer guide

Paths are relative to the repository root (directory containing **`pom.xml`**).

## Release lines

**Documentation convention:** README, CONTRIBUTING, CHANGELOG, and integrator-facing text use **version numbers only** — never Git branch names.

| Version | Java | APIs | `Service` stubs |
| ------- | ---- | ---- | ----------------- |
| **1.6.4.1** | 8 | **`javax.xml.ws`**, **`javax.xml.bind`** | **14** (standard HI B2B) |
| **1.7.1.1** | 11 | **Jakarta** XML WS / Bind | **26** (full MCA) |

**Git branch mapping (maintainers / checkout only - do not use in integrator docs):**

| Version | Official Git branch |
| ------- | ------------------- |
| **1.6.4.1** | `master` |
| **1.7.1.1** | `java-11` |

**Do not merge this line into `master`.** `master` stays **1.6.4.1**. This line is Git **`java-11`** / Maven **1.7.1.1**.

**This 1.7.1.1 line (`1.7.1.1-SNAPSHOT`):** Java **11**, committed **Jakarta** generated types, **26** primary HI B2B **`@WebServiceClient`** services. **`hi-b2b-client`** **1.7.1.1** uses **`hi-wsdl`** at the **same version**.

## Artifact

- **`au.gov.nehta:hi-wsdl`** — HI WSDL on the classpath + pre-generated JAX-WS/JAXB types. Sibling lines share this coordinate and differ by **version**.
- **Not included:** facade clients, TLS/signing, or runtime filesystem WSDL resolution (**`HiWsdlArtifactRoot`** lives in **hi-b2b-client-java**).

## Layout

| Path | Role |
| ---- | ---- |
| `src/main/resources/` | HI `HI_*.wsdl` (classpath root; packaged in JAR) |
| `src/main/java/` | Committed generated types + `hi_override` XMLDSig |
| `src/test/java/au/gov/nehta/hiwsdl/` | Offline binding smoke tests |
| `wsdls/readme.txt` | Licensed WSDL tree staging instructions (tracked) |
| `wsdls/xml/` | Local licensed WSDL/XSD for regeneration (**gitignored**) |
| `pom.xml` (`-Pregenerate-sources`) | **26** **`wsimport`** executions; align with **hi-b2b-client-java** **1.7.1.1** when adding services |
| `.github/workflows/ci.yml` | GitHub Actions **`mvn verify`** on **`master`** and **`java-11`** |

## HI service coverage (`1.7.1.1`)

**26** primary HI B2B interfaces (consumer 3.0–4.0, provider 3.2 / 5.0 / 5.1, TDS 5.1, batch async 5.1, etc.). Interface-only WSDL variants may also exist under **`src/main/resources`**.

**ProviderMatchProviderAdministrativeIndividual** is intentionally out of scope (virtual service; not a published HI B2B interface in this artifact).

## Build (`1.7.1.1` line)

- **`maven.compiler.release` 11**
- Compile deps: **`jakarta.xml.bind-api` 4.0.5**, **`jakarta.xml.ws-api` 4.0.3** — no **`jaxws-rt`** in the published JAR (**`test`** scope only, for **`GeneratedWsdlBindingsTest`**)
- **`jaxws-rt` 4.0.4** for **`-Pregenerate-sources`** only (via **`jaxws-tools`** on **`jaxws-maven-plugin`** classpath; not a compile/runtime dependency of the published JAR)
- Generated sources are **committed**; root POM has **no** default **`wsimport`** execution
- **`maven-enforcer-plugin`:** bans legacy Metro **`webservices-*`** and **`javax.xml.ws` / `javax.xml.bind`**
- **`maven-gpg-plugin`:** skipped unless **`-Dgpg.skip=false`**
- **`maven-javadoc-plugin`:** **`doclint=none`**, **`verbose=false`**, **`quiet=true`**, **`failOnWarnings=false`**, **`detectOfflineLinks=false`**, **`source=11`**. Do not hand-edit Javadoc in generated **`src/main/java`**.
- **Regenerate committed types:** `mvn -B clean -Pregenerate-sources generate-sources process-sources "-Dhi.wsdl.sync.generated=true"` — licensed tree at **`hi.wsdl.tree.root`** (default **`wsdls/xml/`**). Copy updated **`HI_*.wsdl`** into **`src/main/resources/`** when interfaces change. Pins **`jaxb-xjc`** **4.0.9** on **`jaxws-maven-plugin`**.

Align **`jaxws-rt`** with **hi-b2b-client-java** when bumping toolchain versions.

## Release

Publishing uses **`central-publishing-maven-plugin`** (Sonatype Central Portal). Copy **`settings.xml.example`** → **`settings.xml`**, server id **`central`**.

**Parallel release lines (maintainers only):** each Git branch publishes a **different Maven version** — integrators choose by coordinate, not branch name.

| Official branch | Java | HI client / WSDL version | Facades |
| --------------- | ---- | ------------------------ | ------- |
| **`master`** | 8 / javax | **1.6.4.1** | 14 |
| **`java-11`** | 11 / Jakarta | **1.7.1.1** | 26 |

Release **`hi-wsdl`** and **`hi-b2b-client`** at the **same GA version** on the matching pair before integrators upgrade.

### SNAPSHOT or manual GA

1. Update **CHANGELOG.md** (and **`pom.xml`** / SCM **`<tag>`** for manual GA).
2. **`mvn -B "-Prelease" clean verify`**
3. **`mvn -B "-Prelease" deploy`**

Git/SCM settings for **`maven-release-plugin`** live in **`pom.xml`** properties (**`scm.repo.url`**, **`release.*`**). Tags default to **`{artifactId}-{version}`** (e.g. **`hi-wsdl-1.7.1.1`**).

### Automated GA (`maven-release-plugin`)

Run on the **target branch** with a **clean** working tree. The plugin commits version bumps, creates the release tag, deploys from the tag checkout, bumps to the next **`-SNAPSHOT`**, and **pushes branch + tag** (**`pushChanges`** / **`remoteTagging`** in **`pom.xml`**). Git remote credentials (SSH or HTTPS) must work non-interactively.

```text
mvn -B "-Prelease" release:prepare release:perform -DreleaseVersion=1.7.1.1 -DdevelopmentVersion=1.7.1.2-SNAPSHOT -Dtag=hi-wsdl-1.7.1.1
```

Cut **`hi-wsdl`** and **`hi-b2b-client`** at the **same** GA (this line: **`hi-wsdl-1.7.1.1`** and **`hi-b2b-client-1.7.1.1`**). Omit **`-D...`** only if you accept interactive prompts.

**After success:** confirm the artifact on Central; repeat on the paired client repo. No extra Git steps unless push failed (then **`git push origin java-11`** and **`git push origin <tag>`**).

**`-Dgpg.skip=false`** is equivalent to **`-Prelease`** for signing.

## Copyright

Copyright 2012 NEHTA. Copyright 2021-2026 ADHA. Apache License 2.0 — see **LICENSE.txt**.
