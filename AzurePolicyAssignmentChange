# This script uses the API to force a policy compliance check at the subscription level
param (
    [string] $SubscriptionId = "xxxxxxxxx",
    [string] $SourceRepo = "https://raw.githubusercontent.com/lukaszboduch/SecurityCenterPolicy/master/",
    [string] $ASCpolicy = "body_input.json",
    [string] $PolicyDefinitionID = "/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8"
)

# Import modules
Import-Module Az.Resources
Import-Module Az.Security

# Connect to Azure subscription
Connect-AzAccount

# Get the Policy assignment name based on Policy Definition ID 
$Policy = Get-AzPolicyAssignment -policydefinitionid $PolicyDefinitionID
$PolicyName = $Policy.Name
$PolicyDisplayName = $Policy.properties.displayname
$PolicyScope = $Policy.properties.scope

# Gets the latest valid access token for API call
$accessToken = ((Get-AzContext).TokenCache.ReadItems() | Where { $_.TenantId -eq (Get-AzContext).Tenant -and $_.Resource -eq "https://management.core.windows.net/" } | Sort-Object -Property ExpiresOn -Descending)[0].AccessToken
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $accessToken
}

# Get the Body JSON content
$BodyInputUri = $SourceRepo + $ASCpolicy
$Body = Invoke-WebRequest -Uri $BodyInputUri
$Body = $Body.Content

# Full subscription invoke
$restUri = "https://management.azure.com/subscriptions/$subscriptionId/providers/Microsoft.Authorization/policyAssignments/$PolicyName/?api-version=2018-05-01" 
Invoke-RestMethod -Uri $restUri -Method PUT -Headers $authHeader -Body $Body -OutFile $Outfile
$Outfile | ConvertTo-Json | Set-Content .\Out.json

# Set the Policy assignment Display Name
Set-AzPolicyAssignment -Name $Policyname -Scope $PolicyScope -DisplayName $PolicyDisplayName



