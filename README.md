# MuleSoft CI/CD — CloudHub 2.0 Deployment

[![DEV Deploy](https://github.com/Mulejoy/mulesoft-ci-cd/actions/workflows/dev-deploy.yml/badge.svg)](https://github.com/Mulejoy/mulesoft-ci-cd/actions/workflows/dev-deploy.yml)
[![TEST Deploy](https://github.com/Mulejoy/mulesoft-ci-cd/actions/workflows/test-deploy.yml/badge.svg)](https://github.com/Mulejoy/mulesoft-ci-cd/actions/workflows/test-deploy.yml)
[![PROD Deploy](https://github.com/Mulejoy/mulesoft-ci-cd/actions/workflows/prod-deploy.yml/badge.svg)](https://github.com/Mulejoy/mulesoft-ci-cd/actions/workflows/prod-deploy.yml)

GitHub Actions CI/CD pipeline for deploying MuleSoft applications to **CloudHub 2.0** via the Mule Maven Plugin. Supports automated deployment to DEV, TEST, and PROD using Connected App authentication.

## Prerequisites

- **Anypoint Platform** account with CloudHub 2.0 access
- **Java 17** (local builds and CI runners)
- An **Anypoint Platform Connected App** with the following scopes:
  - `Exchange Contributor`
  - `CloudHub Network Administrator`
- **GitHub Secrets** configured in your repository:
  - `CONNECTED_APP_CLIENT_ID`
  - `CONNECTED_APP_CLIENT_SECRET`

## Project Structure

```
mulesoft-ci-cd/
├── .github/
│   └── workflows/
│       ├── dev-deploy.yml       # Push to dev  → deploys to DEV environment
│       ├── test-deploy.yml      # PR to test   → deploys to TEST environment
│       └── prod-deploy.yml      # PR to main   → deploys to PROD environment
├── exchange-docs/
│   └── home.md                  # Anypoint Exchange documentation
├── src/
│   ├── main/
│   │   ├── mule/                # Mule application flows
│   │   └── resources/           # Properties and configuration files
│   └── test/
│       └── resources/           # MUnit test resources
├── mule-artifact.json           # Mule artifact metadata
├── pom.xml                      # Maven build and deployment configuration
└── README.md
```

> **Note**: `src/` is created by Anypoint Studio when you add your first Mule flow. The repository ships without it as a CI/CD scaffold.

## Environment Setup

### 1. Create a Connected App

1. Log in to Anypoint Platform → **Access Management → Connected Apps**
2. Create a new Connected App (Credentials type: **Client Credentials**)
3. Assign scopes: `Exchange Contributor`, `CloudHub Network Administrator`
4. Save the Client ID and Client Secret

### 2. Add GitHub Secrets

In your repository: **Settings → Secrets and variables → Actions → New repository secret**

| Secret | Description |
|--------|-------------|
| `CONNECTED_APP_CLIENT_ID` | Connected App Client ID |
| `CONNECTED_APP_CLIENT_SECRET` | Connected App Client Secret |

### 3. Configure GitHub Environments (Optional but Recommended)

Under **Settings → Environments**, create three environments:

| Environment | Protection |
|-------------|-----------|
| `development` | None |
| `test` | Required reviewers |
| `production` | Required reviewers + deployment branch `main` only |

## Branching Strategy

```
main          ← Production releases (PR from test)
└── test      ← Test/UAT (PR from dev)
    └── dev   ← Integration (push directly)
        └── feature/*  ← Feature development
```

| Branch | Trigger | Target Environment | Approval |
|--------|---------|-------------------|----------|
| `dev` | Push | DEV | None |
| `test` | Pull Request | TEST | Recommended |
| `main` | Pull Request | PROD | Required |

## Local Development

```bash
# Compile only
mvn clean compile

# Run MUnit tests
mvn test

# Package into deployable JAR
mvn package

# Deploy to a specific environment
mvn deploy -DmuleDeploy \
  -Denvironment=DEV \
  -Dclient.id=<your-client-id> \
  -Dclient.secret=<your-client-secret>
```

## Maven Configuration

### Environment Profiles (`pom.xml`)

| Profile | `-Denvironment` | Application Name | CloudHub Target |
|---------|----------------|-----------------|----------------|
| `dev` | `DEV` | `mulesoft-ci-cd-dev` | `Cloudhub-US-East-2` |
| `test` | `TEST` | `mulesoft-ci-cd-test` | `Cloudhub-US-East-2` |
| `prod` | `PROD` | `mulesoft-ci-cd` | `Cloudhub-US-East-2` |

### Key Versions

| Component | Version |
|-----------|---------|
| Mule Runtime | 4.9.7 |
| Java | 17 |
| HTTP Connector | 1.10.3 |
| Sockets Connector | 1.2.5 |
| Mule Maven Plugin | 4.1.1 |

### Authentication in `settings.xml`

Credentials are passed as Maven system properties and interpolated into `settings.xml` at deploy time — no secrets are written to disk:

```xml
<server>
  <id>Repository</id>
  <username>~~~Client~~~</username>
  <password>${client.id}~?~${client.secret}</password>
</server>
```

## Troubleshooting

### 401 Unauthorized
```
401 Unauthorized: Missing credentials
```
**Fix**: Verify `CONNECTED_APP_CLIENT_ID` and `CONNECTED_APP_CLIENT_SECRET` secrets are set correctly and the Connected App has the `Exchange Contributor` and `CloudHub Network Administrator` scopes.

### Application Not Found / Target Not Available
```
Could not find target or environment
```
**Fix**: Confirm the CloudHub 2.0 target (`Cloudhub-US-East-2`) is available in the target Anypoint environment and that the Connected App has access to that Business Group.

### Dependency Resolution Failure
```
Could not resolve artifact
```
**Fix**: Confirm the Anypoint Exchange repository is accessible and the Connected App has the `Exchange Contributor` scope.

### Profile Not Activating
```
deployment.appName is not set
```
**Fix**: Ensure `-Denvironment=DEV`, `-Denvironment=TEST`, or `-Denvironment=PROD` is passed exactly (case-sensitive) to trigger the correct Maven profile.

## Security Notes

- All credentials are stored as GitHub Secrets — never commit them to the repository
- `settings.xml` uses Maven property interpolation (`${client.id}`) so secrets are only present in memory during the Maven process
- GitHub environment protection rules provide an additional approval gate before production deployments
- Rotate Connected App credentials periodically via Anypoint Platform → Access Management

## References

- [MuleSoft CloudHub 2.0 Deployment](https://docs.mulesoft.com/cloudhub-2/ch2-deploy-private-space)
- [Mule Maven Plugin](https://docs.mulesoft.com/mule-runtime/4.4/mmp-concept)
- [Anypoint Connected Apps](https://docs.mulesoft.com/access-management/connected-apps-overview)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

## License

This project is intended as an internal CI/CD template. Refer to your organization's licensing policy for distribution and reuse terms.
