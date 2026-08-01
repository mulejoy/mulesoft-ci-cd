# mulesoft-ci-cd

A reference Mule 4 application demonstrating a complete CI/CD pipeline for **CloudHub 2.0** using GitHub Actions and the Mule Maven Plugin. Use this as a starting template for any MuleSoft project that needs automated build, test, and multi-environment deployment.

---

## Prerequisites

| Tool | Version |
|------|---------|
| Java (JDK) | 17 |
| Maven | 3.9+ |
| Anypoint Studio _(optional, local dev)_ | 7.x |
| Anypoint Platform account | — |

---

## Environment Setup

1. **Create a Connected App** in Anypoint Platform:
   - Navigate to **Access Management → Connected Apps**
   - Grant scopes: `Design Center Developer`, `Exchange Contributor`, `CloudHub Network Administrator`

2. **Copy `.env.example` to `.env`** and fill in your credentials (never commit `.env`):
   ```bash
   cp .env.example .env
   ```

3. **Create a Maven `settings.xml`** with the Anypoint server entry (Maven reads this at deploy time):
   ```xml
   <settings>
     <servers>
       <server>
         <id>Repository</id>
         <username>${env.ANYPOINT_CLIENT_ID}</username>
         <password>${env.ANYPOINT_CLIENT_SECRET}</password>
       </server>
     </servers>
   </settings>
   ```

4. **Add GitHub Secrets** (for CI):
   - `ANYPOINT_CLIENT_ID`
   - `ANYPOINT_CLIENT_SECRET`

---

## Installation & Quickstart

```bash
# Clone
git clone https://github.com/<org>/mulesoft-ci-cd.git
cd mulesoft-ci-cd

# Build and package (skips deploy)
mvn clean package

# Run locally via Anypoint Studio or embedded runtime
# The app starts on http://localhost:8081
# Test endpoint: GET http://localhost:8081/hello
```

### Deploy to CloudHub 2.0

```bash
# Deploy to DEV (default)
mvn deploy -DmuleDeploy

# Deploy to a specific environment and target
mvn deploy -DmuleDeploy \
  -Ddeployment.env=PROD \
  -Ddeployment.target=Cloudhub-US-East-2
```

---

## CI/CD Pipeline

The GitHub Actions workflow at `.github/workflows/deploy.yml` runs automatically:

| Trigger | Job |
|---------|-----|
| Pull request → `main` | Build & package only |
| Push → `main` | Build + deploy to **DEV** |
| Push → `release/**` | Build + deploy to **DEV** → **PROD** (with environment approval gate) |

---

## Folder Structure

```
mulesoft-ci-cd/
├── .github/
│   └── workflows/
│       └── deploy.yml          # CI/CD pipeline definition
├── exchange-docs/
│   └── home.md                 # Anypoint Exchange documentation page
├── src/
│   ├── main/
│   │   ├── mule/
│   │   │   └── mulesoft-ci-cd.xml   # Mule flows
│   │   └── resources/
│   │       ├── application-types.xml # DataWeave type catalog
│   │       └── log4j2.xml            # Runtime logging config
│   └── test/
│       └── resources/
│           └── log4j2-test.xml       # Test logging config (console output)
├── .env.example                # Required environment variable template
├── .gitignore
├── mule-artifact.json          # Mule runtime metadata
└── pom.xml                     # Maven build + CloudHub 2.0 deployment config
```

---

## Key Configuration (pom.xml)

| Property | Default | Override |
|----------|---------|---------|
| `deployment.env` | `DEV` | `-Ddeployment.env=PROD` |
| `deployment.target` | `Cloudhub-US-East-2` | `-Ddeployment.target=<target>` |
| `app.runtime` | `4.9.7:23e-java17` | Update in `pom.xml` |

---

## License

This project is released under the [MIT License](LICENSE).

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-change`
3. Commit your changes and open a pull request against `main`
4. The CI pipeline will run automatically on your PR
