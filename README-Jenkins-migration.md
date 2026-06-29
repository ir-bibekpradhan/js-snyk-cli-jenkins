# Jenkins conversion for JS-SNYK-cli-STAGE

This package contains a Jenkinsfile converted from the uploaded repository's GitHub Actions workflows.

## Main workflows found

- `.github/workflows/InvisiRisk.yml`
  - Manual workflow
  - InvisiRisk PSE setup
  - Checkout
  - Node.js 20
  - `npm install --legacy-peer-deps`
  - `npm ls`

- `.github/workflows/irhaha.yml`
  - InvisiRisk PSE setup
  - Docker BuildKit build
  - Uses `PSE_CA_CERT_PATH` and `PSE_PROXY_IP` from PSE setup
  - Extracts `/app/archive.zip` from the image
  - Uploads artifact

- `.github/workflows/fedora.yml` and `.github/workflows/lolololol.yml`
  - OS/architecture variants of the same npm install flow

- Other workflows such as OSV scan, Danger, docs sync, and IaC smoke tests should usually be separate Jenkins jobs because they have different triggers and credentials.

## Jenkins credentials required

Create this Jenkins credential:

| Jenkins Credential ID | Type | Used for |
|---|---|---|
| `IR_API_KEY` | Secret text | InvisiRisk PSE bootstrap token |

In Jenkins:

```text
Manage Jenkins -> Credentials -> System -> Global credentials -> Add Credentials
```

Use:

```text
Kind: Secret text
ID: IR_API_KEY
Secret: <your InvisiRisk API key>
```

## Jenkins job setup

Use a Pipeline job or Multibranch Pipeline:

```text
New Item -> Pipeline
Pipeline script from SCM
SCM: Git
Repository URL: <your GitHub repo URL>
Branch: main
Script Path: Jenkinsfile
```

For GitHub push triggers, add a GitHub webhook:

```text
http://<your-jenkins-url>/github-webhook/
```

## Jenkins requirements

The generated Jenkinsfile uses a Docker agent:

```groovy
agent {
    docker {
        image 'node:20-bookworm'
        args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
    }
}
```

So Jenkins needs:

- Docker available on the Jenkins agent
- Jenkins Docker Pipeline plugin
- Permission for the Jenkins agent to access `/var/run/docker.sock`

If your Jenkins agent already has Node.js 20, npm, Git, curl, and Docker installed, you can replace the `agent { docker { ... } }` block with:

```groovy
agent any
```

## Important conversion note

GitHub Actions can use `uses: invisirisk/pse-action@latest`. Jenkins cannot use GitHub Actions directly, so the Jenkinsfile replaces it with the raw bootstrap call:

```bash
curl -sSf -H "x-api-key: ${IR_TOKEN}" \
  "${IR_URL}/ingestionapi/v1/pse/bootstrap?mode=native&runner=any" \
  -o /tmp/bootstrap.sh
bash /tmp/bootstrap.sh
```

Every later shell stage sources `/tmp/ir_envs` again because Jenkins starts a new shell for each `sh` block.

## Artifact behavior

The GitHub `actions/upload-artifact` step was converted to:

```groovy
archiveArtifacts artifacts: 'archive.zip', fingerprint: true, onlyIfSuccessful: true
```
