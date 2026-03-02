# Publishing to GitHub Packages

The project is set up to publish Maven artifacts to [GitHub Packages](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry).

## 1. Maven profile

The root `pom.xml` includes a **`github-packages`** profile with `distributionManagement` for the GitHub registry. The server id used for authentication is **`github`**.

## 2. Authentication

### Local publishing

Add a server entry with the same `id` as in the profile to `~/.m2/settings.xml`:

```xml
<settings>
  <servers>
    <server>
      <id>github</id>
      <username>YOUR_GITHUB_USERNAME</username>
      <password>YOUR_GITHUB_TOKEN</password>
    </server>
  </servers>
</settings>
```

- **username** — your GitHub login (or `GITHUB_ACTOR` in CI).
- **password** — [Personal Access Token (classic)](https://github.com/settings/tokens) with `write:packages` and `read:packages` (when publishing from another repo, `repo` may be required).

Do not commit `settings.xml` containing the token to the repository.

### CI (GitHub Actions)

Set the environment variables in your workflow and use the built-in `GITHUB_TOKEN`:

```yaml
env:
  GITHUB_REPOSITORY_OWNER: ${{ github.repository_owner }}
  GITHUB_REPOSITORY_NAME: ${{ github.event.repository.name }}
  # for deploy the server id=github must use GITHUB_TOKEN:
  # add a step that writes settings.xml or use actions/setup-java with server-id
```

Alternatively, pass the owner and repository via Maven properties:

```bash
mvn deploy -P github-packages \
  -Dgithub.packages.owner=YOUR_USER \
  -Dgithub.packages.repo=YOUR_REPO \
  -DskipTests
```

## 3. Publishing artifacts

Specify the owner and repository (if not set via env) and run deploy:

```bash
# Core and generator only (library)
mvn deploy -P github-packages \
  -Dgithub.packages.owner=YOUR_GITHUB_USERNAME \
  -Dgithub.packages.repo=YOUR_REPO \
  -pl modules/openapi-generator-core,modules/openapi-generator -am

# Including CLI (fat JAR)
mvn deploy -P github-packages \
  -Dgithub.packages.owner=YOUR_GITHUB_USERNAME \
  -Dgithub.packages.repo=YOUR_REPO \
  -pl modules/openapi-generator-core,modules/openapi-generator,modules/openapi-generator-cli -am
```

Replace `YOUR_GITHUB_USERNAME` and `YOUR_REPO` with your values. **Use the short repository name only** (e.g. `openapi-generator`), not the full URL (e.g. not `https://github.com/owner/openapi-generator`).

## 4. Consuming artifacts from GitHub Packages

In a project that depends on your `openapi-generator`, add the repository and dependency:

**pom.xml:**

```xml
<repositories>
  <repository>
    <id>github</id>
    <url>https://maven.pkg.github.com/YOUR_USER/YOUR_REPO</url>
  </repository>
</repositories>

<dependencies>
  <dependency>
    <groupId>org.openapitools</groupId>
    <artifactId>openapi-generator</artifactId>
    <version>7.21.0-SNAPSHOT</version>
  </dependency>
</dependencies>
```

For private packages, configure a server with `id=github` and a token with `read:packages` in the same `settings.xml`.

---

## Troubleshooting

**Deploy URL looks like `.../owner/https://github.com/owner/repo`**  
You passed the full repository URL as `github.packages.repo`. Use only the repository name (e.g. `openapi-generator`), not `https://github.com/owner/openapi-generator`.
