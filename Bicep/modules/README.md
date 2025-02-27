# Azure-Policy-As-Code/Bicep/Modules

Learning resources :books:
* [https://docs.microsoft.com/en-us/azure/governance/policy/overview](https://docs.microsoft.com/en-us/azure/governance/policy/overview)
* [policy definitions](https://docs.microsoft.com/en-us/azure/templates/microsoft.authorization/policydefinitions?tabs=bicep)
* [policyset definitions (initiatives)](https://docs.microsoft.com/en-us/azure/templates/microsoft.authorization/policysetdefinitions?tabs=bicep)
* [policy assignments](https://docs.microsoft.com/en-us/azure/templates/microsoft.authorization/policyassignments?tabs=bicep)

## Blogs that might interest you

* [Global Azure: Policy as Code with Bicep for Enterprise Scale](https://jloudon.com/cloud/Global-Azure-Policy-as-Code-with-Bicep-for-Enterprise-Scale/)
* [Azure Spring Clean: DINE to Automate your Monitoring Governance with Azure Monitor Metric Alerts](https://jloudon.com/cloud/Azure-Spring-Clean-DINE-to-Automate-your-Monitoring-Governance-with-Azure-Monitor-Metric-Alerts/)
* [Cloud Governance with Azure Policy Part 1](https://jloudon.com/cloud/Cloud-Governance-with-Azure-Policy-Part-1/)
* [Cloud Governance with Azure Policy Part 2](https://jloudon.com/cloud/Cloud-Governance-with-Azure-Policy-Part-2/)

### Authored & Tested with

* [azure-cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) version 2.20.0
* bicep cli version 0.3.255 (589f0375df)
* [bicep](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep) 0.3.1 vscode extension

### Example Deployment Steps

```s
# optional step to view the JSON/ARM template
az bicep build -f ./main.bicep

# required steps
az login
az deployment sub create -f ./main.bicep -l eastus --confirm-with-what-if

# optional step to trigger a subscription-level policy compliance scan 
az policy state trigger-scan --no-wait
```

> Note regarding resources with dependencies on other resources e.g. Bicep role assignments for new service principals (SP) created by policy assignments will sometimes fail to find the new SP upon 1st run (even though a dependency exists between the resources). This role assignment failure will not reoccur upon a 2nd run of the main.bicep file. The same can apply for policy assignments, policy initiatives, and policy definitions.

### GitHub Actions Workflows

* **Bicep-CI.yml**
* ensures .bicep files are building successfully to ARM/JSON

```yaml
name: Bicep-CI
on:
  push:
    branches: [ main ]
    paths:
    - '**.bicep'
  pull_request:
    branches: [ main ]
    paths:
    - '**.bicep'
  schedule:
  - cron: "0 0 * * *" #at the end of every day
    
jobs:

  Bicep-CI:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Bicep CI
      uses: aliencube/bicep-build-actions@v0.1
      with:
        files: ./main.bicep
```

* **Bicep-CD.yml**
* ensures .bicep files are deploying successfully to 3x Azure environments (DevTest,NonProd,Prod)

```yaml
name: Bicep-CD
on:
  push:
    branches: [ main ]
    paths:
    - '**.bicep'
  workflow_dispatch:
  schedule:
  - cron: "0 0 * * *" #at the end of every day
    
jobs:

  DEVTEST-BICEP-CD:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS_DEVTEST }}
    - name: Bicep CD
      id: bicepCD
      continue-on-error: true
      uses: azure/CLI@v1
      with:
        inlineScript: |
          az deployment sub create -n devtest-bicep-cd -f ./main.bicep -l eastus -o none
    - name: Sleep for 30s
      if: ${{ steps.bicepCD.outcome == 'failure' && steps.bicepCD.conclusion == 'success' }}
      uses: juliangruber/sleep-action@v1
      with:
        time: 30s
    - name: Bicep CD Retry
      if: ${{ steps.bicepCD.outcome == 'failure' && steps.bicepCD.conclusion == 'success' }}
      uses: azure/CLI@v1
      with:
        inlineScript: |
          az deployment sub create -n devtest-bicep-cd -f ./main.bicep -l eastus -o none

  NONPROD-BICEP-CD:
    needs: DEVTEST-BICEP-CD
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS_NONPROD }}
    - name: Bicep CD
      id: bicepCD
      continue-on-error: true
      uses: azure/CLI@v1
      with:
        inlineScript: |
          az deployment sub create -n nonprod-bicep-cd -f ./main.bicep -l eastus -o none
    - name: Sleep for 30s
      if: ${{ steps.bicepCD.outcome == 'failure' && steps.bicepCD.conclusion == 'success' }}
      uses: juliangruber/sleep-action@v1
      with:
        time: 30s
    - name: Bicep CD Retry
      if: ${{ steps.bicepCD.outcome == 'failure' && steps.bicepCD.conclusion == 'success' }}
      uses: azure/CLI@v1
      with:
        inlineScript: |
          az deployment sub create -n nonprod-bicep-cd -f ./main.bicep -l eastus -o none

  PROD-BICEP-CD:
    needs: NONPROD-BICEP-CD
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS_PROD }}
    - name: Bicep CD
      id: bicepCD
      continue-on-error: true
      uses: azure/CLI@v1
      with:
        inlineScript: |
          az deployment sub create -n prod-bicep-cd -f ./main.bicep -l eastus -o none
    - name: Sleep for 30s
      if: ${{ steps.bicepCD.outcome == 'failure' && steps.bicepCD.conclusion == 'success' }}
      uses: juliangruber/sleep-action@v1
      with:
        time: 30s
    - name: Bicep CD Retry
      if: ${{ steps.bicepCD.outcome == 'failure' && steps.bicepCD.conclusion == 'success' }}
      uses: azure/CLI@v1
      with:
        inlineScript: |
          az deployment sub create -n prod-bicep-cd -f ./main.bicep -l eastus -o none
```