# @juptierone/action-transponder Documentation

action-transponder will run the JupiterOne Transponder package using the specified config file and upload the results to a JupiterOne account. The following configurations are required for setup

## transponder.yml 
### create a config-transponder.yml at the root of the code repo

```yaml 
plugins:
  - core=@jupiterone/transponder-core-plugin

# Collect data
collect:
  # Populate some default metadata properties for the root entity
  # (including `dateIngested`)
  root:
    collector: core:root
    params:
      _key: root
      _type: project_metadata
      _class: ProjectMetadata

  # Each collection command has a unique name
  git-remote-origin:
    collector: core:git-remote-origin
    failOnError: false
    params:
      # What is the name of the remote origin? (typically "origin")
      remoteName: origin

  codeowners:
    collector: 'core:codeowners'
    params:
      file: <rootDir>/CODEOWNERS
    failOnError: false

  docker-file:
    collector: "core:docker-file"
    params:
      file: <rootDir>/Dockerfile

publish:
  j1:
    publisher: 'core:j1'
    params:
      # NOTE:
      # Tokens will be replaced from environment variables with matching name.
      apiKey: ${env.J1_API_KEY}
      apiDomain: ${env.J1_API_DOMAIN}
      accountId: ${env.J1_ACCOUNT_ID}
```

## .github/workflows/transponder.yml
### create a transponder.yml at the root of the code repo
It is recommended to include both a "push on branch" as well as a "schedule" task. This will ensure that trasponder runs when code changes as well as regularly so the data in JupiterOne is up to date. 

```yaml
name: 'Transponder'

on:
  push:
    branches:
      - main
  schedule:
    - cron:  '10 1 * * 1'

jobs:
  transponder:
    runs-on: ubuntu-latest
    name: Run transponder
    steps:
      - name: Check out repo
        uses: actions/checkout@v3
      - name: Transponder Update
        uses: jupiterone/action-transponder@main
        with:
          j1_api_key: ${{ secrets.J1_API_KEY }}
          j1_api_domain: ${{ secrets.J1_API_DOMAIN }}
          j1_account_id: ${{ secrets.J1_ACCOUNT_ID }}
          transponder_config: config-transponder.yml
```

## Add 'Repository Secrets' to the repo or organization
J1_ACCOUNT_ID: The 'Account ID' field can be found in the 'Account Management' page of your JupiterOne account (requires Admin privileges).  

J1_API_DOMAIN: This should be "api.us.jupiterone.io" unless otherwise specified.

J1_API_KEY: Create an API key in JupiterOne (requires Admin privileges).