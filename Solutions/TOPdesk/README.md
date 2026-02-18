# TOPdesk Data Connector

<img src="https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/Logos/TOPdesk.svg" alt="TOPdesk" width="20%"/><br>

This solution ingests **TOPdesk ITSM Incidents** into Microsoft Sentinel using the [Codeless Connector Framework](https://learn.microsoft.com/en-us/azure/sentinel/create-codeless-connector).

## Overview

[TOPdesk](https://www.topdesk.com/en/itsm-software/) helps busy service teams handle incoming requests, automatically assign tasks to the right people, and see who's doing what — so you can stay ahead instead of just keeping up.

This connector polls the TOPdesk REST API for incidents and stores them in your Log Analytics workspace for monitoring, analysis, and threat detection within Microsoft Sentinel.

## Authentication

This connector uses **Basic authentication** (operator username + application password).

You will need to provide the following when configuring the connector:

| Parameter | Description |
|---|---|
| **TOPdesk base URL** | Your TOPdesk tenant hostname (e.g. `yourcompany.topdesk.net`) |
| **Username** | TOPdesk operator username |
| **Application password** | TOPdesk application password for the operator |

## Log ingestion

The connector ingests TOPdesk incidents and stores them in the `TOPdeskIncidents_CL` table with fields including:

- Incident ID, number, title, and description
- Status, priority, impact, and urgency
- Category, subcategory, and call type
- Caller, operator, and operator group details
- Created, updated, closed, and target dates

## Deployment

1. Deploy the solution through the Azure Portal under your Microsoft Sentinel instance.
2. Navigate to **Data Connectors** and open the **TOPdesk** connector page.
3. Enter your TOPdesk base URL, username, and application password.
4. Click **Connect**.

## Verification

After connecting, run the following query in Log Analytics to confirm data is flowing:

```kusto
TOPdeskIncidents_CL
| sort by TimeGenerated desc
| take 10
```

## Support

- **Publisher:** Wortell Enterprise Security
- **Contact:** [mdr@wortell.nl](mailto:mdr@wortell.nl)
- **Website:** [www.wortell.nl](https://www.wortell.nl)

## References

- [TOPdesk API Documentation](https://developers.topdesk.com/tutorial.html)
- [Microsoft Sentinel Documentation](https://learn.microsoft.com/en-us/azure/sentinel/)
- [Codeless Connector Framework](https://learn.microsoft.com/en-us/azure/sentinel/create-codeless-connector)
