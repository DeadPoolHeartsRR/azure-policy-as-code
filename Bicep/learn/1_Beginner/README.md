# Azure-Policy-As-Code/Bicep/Learn/Beginner

* Uses built-in policies
* Uses an initiative and assignment
* 1x main.bicep
* Manual CLI deployment

## Parameters

**Defaults**

```bicep
param policySource string = 'demo/azure-policy-as-code'
param policyCategory string = 'Custom'
param assignmentEnforcementMode string = 'Default'
param listOfAllowedLocations array = [
  'eastus'
  'eastus2'
  'westus'
  'westus2'
]
param listOfAllowedSKUs array = [
  'Standard_B1ls'
  'Standard_B1ms'
  'Standard_B1s'
  'Standard_B2ms'
  'Standard_B2s'
  'Standard_B4ms'
  'Standard_B4s'
  'Standard_D2s_v3'
  'Standard_D4s_v3'
]
```

## Deployment Steps

```s
# optional step to view the JSON/ARM template
az bicep build -f ./main.bicep

# required steps
az login
az deployment sub create -f ./main.bicep -l eastus

# optional step to trigger a subscription-level policy compliance scan 
az policy state trigger-scan --no-wait
```

### Azure Resource Manager (ARM) Template References

* [policy definitions](https://docs.microsoft.com/en-us/azure/templates/microsoft.authorization/policydefinitions?tabs=json)
* [policyset definitions (initiatives)](https://docs.microsoft.com/en-us/azure/templates/microsoft.authorization/policysetdefinitions?tabs=json)
* [policy assignments](https://docs.microsoft.com/en-us/azure/templates/microsoft.authorization/policyassignments?tabs=json)
