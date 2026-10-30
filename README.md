# Lab 7 — Jenkins, Nexus and SonarQube

Repository: https://github.com/dtc235180193-ui/devops-lab7-ci-demo

## Run

```powershell
docker compose up -d
cd sample-java-app
mvn clean test
```

Local services: Jenkins http://localhost:8080, Nexus http://localhost:8081,
SonarQube http://localhost:9000. Ports bind to loopback only. No Docker socket
is exposed to Jenkins because this Java pipeline does not need Docker access.

## Jenkins configuration

- Maven tool: `M3` (Maven 3.9.x, automatic installation).
- Plugins: Pipeline, Git, JUnit, Pipeline Stage View, SonarQube Scanner,
  Config File Provider, Workspace Cleanup, Timestamper, Credentials Binding.
- SonarQube server: `SonarQube`, URL `http://lab7-sonarqube:9000`,
  Secret text credential `sonarqube-token`.
- Nexus Username/password credential: `nexus-credentials`.
- Managed Maven settings: `maven-settings-nexus`, content from
  `maven-settings-nexus.xml`.
- Pipeline script from SCM: GitHub repository, branch `*/main`,
  Script Path **`Jenkinsfile`** (repository root).

The pipeline checks Git for changes every five minutes. GitHub webhooks require
a separately approved public endpoint; localhost is not accessible to GitHub.
Nexus uses `maven-snapshots` for version `1.0.0-SNAPSHOT`; snapshot artifact
filenames are timestamped. JaCoCo generates coverage for SonarQube.

## Stop without deleting data

```powershell
docker compose down
```

Do not add `-v` unless you explicitly want to delete all lab service data.
Credentials under `.secrets/` are local-only: never commit or submit them.

## This machine

The Compose project name `lab7-jenkins-ci` deliberately reuses the existing lab
volumes on this machine. Run Compose commands from this directory rather than
the older copy under Documents, whose configuration exposes additional ports.
Local administrator passwords are in `.secrets/credentials.json` (restricted
to the current Windows user); usernames for all three services are `admin`.
That file and local setup scripts are not part of the public repository or
submission archive. Never run two copies of this stack simultaneously.

Nexus Community 3.97 requires EULA acceptance before repository access and
includes mandatory baseline telemetry. Complete its onboarding wizard before
running the publish stage. The existing SonarQube 9.9.8 image on this machine
is end-of-life and uses an embedded evaluation database: this stack is for the
local lab only, not production or public exposure.
