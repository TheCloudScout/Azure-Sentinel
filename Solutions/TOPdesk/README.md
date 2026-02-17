# TOPdesk — Microsoft Sentinel CCF Solution

This solution ingests **TOPdesk ITSM incidents** into Microsoft Sentinel using the [Codeless Connector Framework (CCF)](https://learn.microsoft.com/en-us/azure/sentinel/create-codeless-connector) with a `RestApiPoller` connector.

## Solution structure

```
Solutions/TOPdesk/
├── createUiDefinition.json                         # Marketplace deployment UI
├── mainTemplate.json                               # Standalone ARM template (manual deploy)
├── SolutionMetadata.json                           # Publisher / offer metadata
├── Data/
│   └── Solution_TOPdeskIncidents.json              # Packaging tool input file
├── Data Connectors/
│   └── TOPdeskIncidents_ccf/
│       ├── TOPdeskIncidents_connectorDefinition.json   # Connector UI definition
│       ├── TOPdeskIncidents_PollerConfig.json           # RestApiPoller configuration
│       ├── TOPdeskIncidents_DCR.json                    # Data Collection Rule
│       └── TOPdeskIncidents_Table.json                  # Custom table schema
└── Package/
    ├── mainTemplate.json                           # Generated packaged ARM template
    ├── createUiDefinition.json                     # Generated packaged UI definition
    └── testParameters.json                         # Test deployment parameters
```

## Prerequisites

| Requirement | Details |
|---|---|
| **PowerShell** | 7.1 or later ([upgrade guide](https://docs.microsoft.com/powershell/scripting/install/migrating-from-windows-powershell-51-to-powershell-7)) |
| **Node.js** | Latest LTS from [nodejs.org](https://nodejs.org/) |
| **YAML module** | `Install-Module powershell-yaml` |
| **ARM TTK** | Clone [arm-ttk](https://github.com/Azure/arm-ttk) to `C:\One` and import the module |
| **Repo clone** | Clone [Azure-Sentinel](https://github.com/Azure/Azure-Sentinel) to `C:\GitHub` (or update `BasePath` in the data input file) |

## Next steps — Packaging

### 1. Verify your data input file

Open `Data/Solution_TOPdeskIncidents.json` and confirm `BasePath` points to your local clone:

```json
"BasePath": "C:\\GitHub\\Azure-Sentinel\\Solutions\\TOPdesk"
```

The `Data Connectors` array must reference the connector definition file:

```json
"Data Connectors": [
    "Data Connectors/TOPdeskIncidents_ccf/TOPdeskIncidents_connectorDefinition.json"
]
```

### 2. Run the V3 packaging tool

```powershell
cd C:\GitHub\Azure-Sentinel\Tools\Create-Azure-Sentinel-Solution\V3

# Catalog mode (default) — uses Microsoft Catalog API for version management
./createSolutionV3.ps1

# When prompted, enter the data folder path:
#   C:\GitHub\Azure-Sentinel\Solutions\TOPdesk\Data

# Or pass it directly:
./createSolutionV3.ps1 -SolutionDataFolderPath "C:\GitHub\Azure-Sentinel\Solutions\TOPdesk\Data"
```

For local development/testing without the Catalog API:

```powershell
# Patch bump (3.0.0 → 3.0.1)
./createSolutionV3.ps1 `
    -SolutionDataFolderPath "C:\GitHub\Azure-Sentinel\Solutions\TOPdesk\Data" `
    -VersionMode "local" `
    -VersionBump "patch"
```

The tool will:
1. Read the connector definition, poller, DCR, and table files
2. Generate `Package/mainTemplate.json` and `Package/createUiDefinition.json`
3. Run ARM TTK validation automatically

### 3. Validate the generated package

#### ARM TTK validation (automatic)

The packaging tool runs this automatically. To re-run manually after edits:

```powershell
cd C:\GitHub\Azure-Sentinel\Solutions\TOPdesk\Package
Test-AzTemplate
```

> **Note:** You can ignore ARM-TTK errors for `contentProductId` / `id` related to *"IDs should be derived from ResourceIds"* — these are known false positives for solution packages.

#### UI validation

1. Open the [CreateUI Sandbox](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/SandboxBlade)
2. Paste the contents of `Package/createUiDefinition.json`
3. Click **Preview** and verify the UI renders correctly

#### Deployment validation

1. Open [Custom Deployment](https://portal.azure.com/#create/Microsoft.Template) in Azure Portal (use `https://aka.ms/AzureSentinelPrP` for preview features)
2. Click **Build your own template in the editor**
3. Paste the contents of `Package/mainTemplate.json`
4. Select your subscription, resource group, and Sentinel workspace
5. Click **Review + Create** and verify the deployment succeeds
6. Navigate to **Data Connectors** in Sentinel and verify the TOPdesk connector appears
7. Open the connector page, fill in the base URL and credentials, and click **Connect**
8. Open browser Developer Tools → **Network** tab to verify the DCR `PUT` request succeeds

### 4. Test data ingestion

After connecting, run in Log Analytics:

```kusto
TOPdeskIncidents_CL
| sort by TimeGenerated desc
| take 10
```

## Standalone deployment (without packaging)

The root `mainTemplate.json` can be deployed directly for testing:

```powershell
az deployment group create \
    --resource-group <your-rg> \
    --template-file mainTemplate.json \
    --parameters workspaceName=<workspace> topdeskBaseUrl=<yourtenant.topdesk.net> username=<user> password=<app-password>
```

## References

- [CCF Packaging Guide](https://github.com/Azure/Azure-Sentinel/blob/master/Tools/Create-Azure-Sentinel-Solution/V3/CCF_README.md)
- [V3 Packaging Tool Guide](https://github.com/Azure/Azure-Sentinel/blob/master/Tools/Create-Azure-Sentinel-Solution/V3/README.md)
- [Create a Codeless Connector](https://learn.microsoft.com/en-us/azure/sentinel/create-codeless-connector)
- [Data Connector UI Definitions Reference](https://learn.microsoft.com/en-us/azure/sentinel/data-connector-ui-definitions-reference)
- [Data Connector Connection Rules Reference](https://learn.microsoft.com/en-us/azure/sentinel/data-connector-connection-rules-reference)
- [Data Collection Rules Overview](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/data-collection-rule-overview)
- [TOPdesk API Documentation](https://developers.topdesk.com/tutorial.html)
